---
layout: post
title: "Distributed iOS builds with Jenkins"
date: 2018-08-03
categories: etc
---

I work on a couple iOS apps on a small team. We collaborate via Bitbucket and
review each others' pull requests before each merge. We wanted a continuous
integration setup that would report build or unit test failures right in each
pull request. We did this with Jenkins. The resulting setup is very simple and
easy to maintain, but I was surprised at how few resources I could find on the
web about it. This is what I did.

## Fastlane

I highly recommend using [Fastlane][fl] for your iOS builds. It makes your
builds much more easily configurable and reproducable and makes it easy to
build, test, and even deploy your app all with one tool. It can be pretty
tricky to set up (especially the app signing), but after many hours of
experimentation I found a very simple setup that worked for us.

[fl]: https://fastlane.tools/

### Signing

First, we didn't use any of Fastlane's recommended signing tools
(match/cert/sigh). I think they're designed for smaller organizations where each
developer is allowed to update signing on their own, where it's not a big deal
to revoke and recreate the signing certificate. But we have an organization-wide
distribution certificate, and most iOS devs don't have access to it because it
would be a huge headache if anyone was to revoke it.

So instead, we just made a new iOS developer account that has the master signing
certificate for the purpose of running on our build machines. Then you just have
to make sure that your Xcode project is set up correctly for that one account to
do signing and you pretty much don't have to do any of Fastlane's fancy signing
stuff. The best way to do this is to open up Xcode on the build machine itself
and go through the steps of pushing a build to TestFlight.

### Config

Here's our config, nice and simple:

{% highlight ruby %}
default_platform :ios

platform :ios do
  before_all do
    # store all output in the Jenkins workspace
    setup_jenkins
  end

  desc 'Run tests'
  lane :test do |opts|
    exception = nil
    %w[TestTarget1 TestTarget2].each do |name|
      begin
        run_tests scheme: name, device: 'iPhone 6', output_types: 'junit', output_files: "#{name}.junit"
      rescue => ex
        exception = ex
      end
    end
    raise exception if exception
  end

  desc 'Build and sign for testflight'
  lane :build do |opts|
    opts[:config] ||= 'Distribution'

    build_app scheme: 'MyApp', configuration: opts[:config], export_xcargs: '-allowProvisioningUpdates'
  end
end
{% endhighlight %}

There's only two tricks here. First because of... reasons, we have multiple test
targets that we run for unit tests. So our testing lane is set up to loop
through them and then fail if any of the tests failed. Second is the
`-allowProvisioningUpdates` option. This allows the build to run unattended in
the case that Xcode is able to automatically fix a provisioning problem (I think
this is a new feature in Xcode 9).

## Jenkins

For the distributed builds, you first need a Jenkins instance that can act as
the master. There are actually tons of guides on how to do this, and there's
nothing specific you need to worry about with respect to iOS or OSX.

We set up each agent to connect via JNLP by having each agent run a .jar that is
provided by the Jenkins master. You need to use [Homebrew][brew] to install Java
8 to run the .jar without issues. Once that's done, it's easy to create a startup
service so that your agent gets restarted on each boot.

[brew]: https://brew.sh

Next set up a multi-branch pipeline job on the master. This way the master can
monitor all of your branches and trigger distributed builds for each one. The
only thing that tripped me up here is that handling submodules requires
"Advanced sub-modules behaviors" so that your submodules update recursively and
use the right credentials.

### Jenksinfile

Configuring the master is the easy part. The hard part is setting up your
pipeline in your Jenkinsfile. This required a ton of trial and error (I feel
like there are no examples of good declarative pipelines anywhere) but the end
result is thankfully uncomplicated:

{% highlight groovy %}
pipeline {
  agent none

  options {
    skipDefaultCheckout true
    ansiColor 'xterm'
  }

  stages {
    stage('Test') {
      agent {
        label 'OSX && xcode'
      }
      environment {
        // needed by Fastlane
        LANG = 'en_US.UTF-8'
        LC_ALL = 'en_US.UTF-8'
        // to find Homebrew's java and bundler
        PATH = "/usr/local/bin:$PATH"
      }
      steps {
        checkout scm
        script {
          // show "build in progress" on bitbucket (using Master's credentials)
          notifyBitbucket credentialsId: '00000000-1111-2222-3333-444444444444'
        }
        sh 'bash install_fastlane.sh'
        sh 'bundle exec fastlane test'
      }
      post {
        always {
          junit '**/output/*.junit'
          script {
            // result is nil on success, which the Stash plugin doesn't handle
            currentBuild.result = currentBuild.result ?: 'SUCCESS'
            notifyBitbucket credentialsId: '00000000-1111-2222-3333-444444444444'
          }
        }
        success {
          deleteDir()
        }
      }
    }
  }
}
{% endhighlight %}

The trick here is that we don't actually need a fully distributed pipeline, we
just want master to hand out jobs to available agents and each agent does the
whole job. For this you need to specify `agent none` and `skipDefaultCheckout
true`, otherwise master will do a full checkout of each branch (pointless! only
the agent needs that) and master will pass your jobs to agents that can't even
perform the build (which essentially DOSes your other build agents if your iOS
build machines get backed up).

As an aside, I don't really understand why Jenkins is designed in this way; our
simple setup of having master delegate builds to agents was _so easy_ to get
wrong. Our initial attempts wasted so much time performing pointless steps and
blocking agents that had no business even touching our iOS app.

To simplify this process, we also configured our app to use bundler to install
gems locally. Our `.bundle/config` is just

{% highlight yaml %}
---
BUNDLE_PATH: ".bundle"
{% endhighlight %}

That way we don't have to worry about setting up gems on each agent, it's
handled entirely in the repository.

We could add `fastlane` to our `Gemfile` but for... reasons, some of our
developers don't want to install it when they build on their own machines. So
instead we have that `install_fastlane.sh` script which runs on the Jenkins
agents

{% highlight bash %}
#!/bin/bash

! grep fastlane Gemfile && echo 'gem "fastlane"' >> Gemfile
bundle install
{% endhighlight %}

## Deploying

You can also do distributed deployments in a similar way. For this you just want
a generic pipeline job. You can add parameters if you want to set deploy
options, for example we have a `buildopts` parameter that is forwarded to the
`build` lane and you can specify what git should checkout with a `ref`
parameter.

{% highlight groovy %}
pipeline {
  agent none

  options {
    skipDefaultCheckout true
    ansiColor 'xterm'
  }

  environment {
    LANG = 'en_US.UTF-8'
    LC_ALL = 'en_US.UTF-8'
    PATH = "/usr/local/bin:$PATH"
    FASTLANE_USER = 'master@email.com'
    FASTLANE_PASSWORD = 'masterpassword'
  }

  stages {
    stage('Deploy') {
      agent {
        label 'OSX && xcode'
      }
      steps {
          deleteDir()
        checkout([$class: 'GitSCM',
            userRemoteConfigs: [[
                url: 'ssh://git@myhost.com:7999/myapp.git',
                credentialsId: 'aaaaaaaa-bbbb-cccc-dddd-eeeeeeeeeeee'
            ]],
            branches: [[name: params.ref]],
            extensions: [[$class: 'SubmoduleOption', parentCredentials: true, recursiveSubmodules: true]]
        ])

        sh 'bash install_fastlane.sh'
        sh "bundle exec fastlane build ${params.buildopts}"
        sh 'bundle exec fastlane run upload_to_testflight ipa:output/MyApp.ipa'
      }
    }
  }
}
{% endhighlight %}

The main differences here are that we have to specify the iTunes Connect account
that is authorized to sign and upload the app and we call fastlane's `build` and
`upload_to_testlight` actions (the former is from our Fastfile, the latter is
built-in).

The tricky part is that hideous checkout block. Since I wanted the git checkout
to be a parameter, I couldn't use the automatic `scm` variable defined by
Jenkins. The pipeline snippet generator is somewhat helpful for figuring out
what syntax to use here, but I still had to clean it up to get this minimal example.

This works great and lets you have (mostly) unattended deploys from whatever
build machine happens to be free. The only part that prevents it from being
fully unattended is that occasionally Apple will force you to log in and answer
the uploading account's security questions.

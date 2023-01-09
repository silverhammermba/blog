---
layout: post
title: "Distributed iOS builds with Jenkins"
date: 2018-08-03
categories: etc
---

#### Updated 2019-02-28

I work on a couple iOS apps on a small team. We collaborate via Bitbucket and
review each others' pull requests before each merge. We wanted a continuous
integration setup that would report build or unit test failures right in each
pull request. We did this with Jenkins. The resulting setup is pretty simple and
easy to maintain, but I was surprised at how few resources I could find on the
web about it. This is what I did.

## Fastlane

**Edit 2019-03-12:** [I don't recommend fastlane anymore][slow].

[slow]: {{ site.baseurl }}{% post_url 2019-03-12-slowlane %}

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

Here's our config, mostly simple:

{% highlight ruby %}
default_platform :ios

platform :ios do
  before_all do
    # store all output in the Jenkins workspace
    # result_bundle: false works around a fastlane bug where run_tests can't find the test results to pass to trainer
    setup_jenkins result_bundle: false
  end

  lane :build_tests do
    run_tests scheme: 'Tests', device: 'iPhone 6', build_for_testing: true, output_types: ''
  end

  lane :test do
    scheme_name = 'Tests'
    run_tests scheme: scheme_name, device: 'iPhone 6', test_without_building: true, output_types: '', fail_build: false
    # because of result_bundle: false, the test results will be in derivedData, but we want them in the output dir
    trainer extension: '.junit', output_directory: ENV['SCAN_OUTPUT_DIRECTORY']
  end

  desc 'Build and sign for testflight'
  lane :build do |opts|
    opts[:config] ||= 'Distribution'

    build_app scheme: 'MyApp', configuration: opts[:config], export_xcargs: '-allowProvisioningUpdates'
  end
end
{% endhighlight %}

We had to make some other changes to get this working. Previously we had
multiple test schemes in our Xcode project (organized by framework), but that
made the fastlane configuration needlessly complex because we had to handle
stuff like aggregating their results and ensuring that all tests were run.
Instead just make a new scheme in Xcode (I called ours "Tests") and add all of
your testing targets to it. Then your Fastfile is cleaner.

We also had to use the [trainer plugin][trainer] to generate the junit report
rather than fastlane itself. It turns out that fastlane uses xcpretty to
generate its junit report and... xcpretty kinda sucks at it. Just look at their
issue tracker: there are **tons** of [open bugs][bugs] related to missing test
results. Though as you can see in the comments of the Fastfile, there were a
couple tricks needed to get fastlane working with trainer...

[trainer]: https://github.com/xcpretty/trainer/tree/master/fastlane-plugin-trainer
[bugs]: https://github.com/xcpretty/xcpretty/issues?utf8=%E2%9C%93&q=is%3Aissue+is%3Aopen+tests

Another trick is the `-allowProvisioningUpdates` option. This allows the build
to run unattended in the case that Xcode is able to automatically fix a
provisioning problem (I think this was a new feature in Xcode 9).

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
    stage('waiting for executor') {
      agent {
        label 'OSX && xcode'
      }
      environment {
        // for fastlane
        LANG = 'en_US.UTF-8'
        LC_ALL = 'en_US.UTF-8'
        // for homebrew
        PATH = "/usr/local/bin:$PATH"
      }
      stages {
        stage("prepare") {
          steps {
            checkout scm
            script {
              // show "build in progress" on bitbucket (using Master's credentials)
              notifyBitbucket credentialsId: 'aaaaaaaa-bbbb-cccc-dddd-eeeeeeeeeeee'
            }
            sh 'bash install_fastlane.sh'
          }
        }
        stage("build") {
          steps {
            sh 'bundle exec fastlane build_tests'
          }
        }
        stage("test") {
          steps {
            sh 'xcrun simctl shutdown all'
            sh 'xcrun simctl erase all'
            sh 'bundle exec fastlane test'
          }
          post {
            always {
              junit 'output/*.junit'
            }
          }
        }
      }
      post {
        always {
          script {
            // result is nil on success, which the Stash plugin doesn't handle
            currentBuild.result = currentBuild.result ?: 'SUCCESS'
            notifyBitbucket credentialsId: 'aaaaaaaa-bbbb-cccc-dddd-eeeeeeeeeeee'
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

The really confusing part here is the nested stages. Notice how that first stage
is called "waiting for executor" even though it contains literally everything?
That's because the `agent { label ... }` part changes everything: this first
stage is running in an `agent none` block, meaning it's running on master
_until_ a suitable agent is found. Then the stage is _done_ because everything
else is running on the agent. Hence the name, because the duration you'll see in
Jenkins is only the time it took master to find an agent to run it on.

The other thing that took me a while to learn the importance of is splitting up
the stages on the agent. Separating prepare/build/test is a huge help for
diagnosing build issues quickly, but also for avoiding pointless errors. For
example, we used to have everything in a single stage and this caused confusing
error messages when some of our build agents started failing while installing
fastlane. Since they never got around to building or testing, there was no junit
report to collect, so the final `post { always { junit` part was _also_ failing
because it couldn't find the report.

The other kinda paranoid thing is those two `simctl` calls. Our app runs
database migrations on launch and we've had issues when multiple branches of
development are using different migrations. This is a problem because the
simulator will actually keep your app around between test runs, so if your agent
is switching between branches with conflicting migrations, xcode might try to
"update" the app on the simulator to a version with a totaly different migration
path, which makes core data very angry. So just to be safe we wipe and restart
everything on the simulator before testing.

To assist running this on distributed agents, we also configured our app to use
bundler to install gems locally. Our `.bundle/config` is just

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

! grep -q fastlane Gemfile &&\
    echo 'gem "fastlane"' >> Gemfile &&\
    echo 'gem "fastlane-plugin-trainer"' >> Gemfile
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

The main differences here are that we have to specify the App Store Connect
account that is authorized to sign and upload the app and we call fastlane's
`build` and `upload_to_testlight` actions (the former is from our Fastfile, the
latter is built-in).

The tricky part is that hideous checkout block. Since I wanted the git checkout
to be a parameter, I couldn't use the automatic `scm` variable defined by
Jenkins. The pipeline snippet generator is somewhat helpful for figuring out
what syntax to use here, but I still had to clean it up to get this minimal example.

This works great and lets you have (mostly) unattended deploys from whatever
build machine happens to be free. Unintuitively, you have to disable two-factor
authentication for the App Store Connect account to make this run unattended. If
you enable two-factor auth, your two-factor session will expire after a month so
someone will have to manually refresh it before deploys wil work again.

---
layout: post
title: "Life in the slow lane"
categories: etc
---

Initially used fastlane for building and testing on jenkins.

Realized this is very heavy-handed. Really bad bugs in test result report.

Proper way would be to use xsl to reformat the plist:
https://warchimede.github.io/2017/07/16/convert-xcode-plist-test-reports-to-junit-xml/

No one knows how to do that. I don't know xsl, I know Ruby.

Don't need general XML parsing, we need to parse only Xcode's
TestSummaries.plist. Don't need general junit output, only need something that
makes sense to Jenkins.

Fastlane is still good: automated screenshots and deployments.

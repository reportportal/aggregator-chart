
# [ReportPortal.io](http://ReportPortal.io)

[![Join Slack chat!](https://img.shields.io/badge/slack-join-brightgreen.svg)](https://slack.epmrpp.reportportal.io/)
[![stackoverflow](https://img.shields.io/badge/reportportal-stackoverflow-orange.svg?style=flat)](http://stackoverflow.com/questions/tagged/reportportal)
[![GitHub contributors](https://img.shields.io/badge/contributors-102-blue.svg)](https://reportportal.io/community)
[![Docker Pulls](https://img.shields.io/docker/pulls/reportportal/service-api.svg?maxAge=25920)](https://hub.docker.com/u/reportportal/)
[![License](https://img.shields.io/badge/license-Apache-brightgreen.svg)](https://www.apache.org/licenses/LICENSE-2.0)
[![Build with Love](https://img.shields.io/badge/build%20with-❤%EF%B8%8F%E2%80%8D-lightgrey.svg)](http://reportportal.io?style=flat)

ReportPortal is a TestOps service, that provides increased capabilities to speed up results analysis and reporting through the use of built-in analytic features.

## Prerequisites

> **Note:** The minimal requirements for a ReportPortal 1-node solution are 2 CPUs and 6Gi of memory

* Kubernetes v1.26+
* Helm Package Manager v3.4+

## Installing the Chart

Add the official ReportPortal Helm Chart repository:

```bash
helm repo add reportportal https://reportportal.io/aggregator && helm repo update aggregator
```

Install the chart:

```bash
helm install my-release reportportal/aggregator
```

## Uninstalling the Chart

```bash
helm uninstall my-release 
```

## Configuration

### Install from sources

For fetching chart dependencies, use the command:

```bash
helm install ./aggregator
```

### Install specific version

To search for available versions of a chart, use:

```bash
helm search repo aggregator --versions
```

To install a specific version of a chart, use:

```bash
helm install my-release reportportal/aggregator --version 1.46.1
```

## Documentation

* [General User Manual](https://reportportal.io/docs/)

## Community / Support

* [**Slack chat**](https://reportportal-slack-auto.herokuapp.com)
* [**Security Advisories**](https://github.com/reportportal/reportportal/blob/master/SECURITY_ADVISORIES.md)
* [GitHub Issues](https://github.com/reportportal/reportportal/issues)
* [Stackoverflow Questions](http://stackoverflow.com/questions/tagged/reportportal)
* [Twitter](http://twitter.com/ReportPortal_io)
* [Facebook](https://www.facebook.com/ReportPortal.io)
* [YouTube Channel](https://www.youtube.com/channel/UCsZxrHqLHPJcrkcgIGRG-cQ)

## License

Report Portal is [Apache 2.0](https://www.apache.org/licenses/LICENSE-2.0).
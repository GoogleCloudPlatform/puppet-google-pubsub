# Google Cloud Pub/Sub Puppet Module

[![Puppet Forge](http://img.shields.io/puppetforge/v/google/gpubsub.svg)](https://forge.puppetlabs.com/google/gpubsub)

#### Table of Contents

1. [Module Description - What the module does and why it is useful](
    #module-description)
2. [Setup - The basics of getting started with Google Cloud Pub/Sub](#setup)
3. [Usage - Configuration options and additional functionality](#usage)
4. [Reference - An under-the-hood peek at what the module is doing and how](
   #reference)
    - [Classes](#classes)
    - [Bolt Tasks](#bolt-tasks)
5. [Limitations - OS compatibility, etc.](#limitations)
6. [Development - Guide for contributing to the module](#development)

## Module Description

This Puppet module manages the resource of Google Cloud Pub/Sub.
You can manage its resources using standard Puppet DSL and the module will,
under the hood, ensure the state described will be reflected in the Google
Cloud Platform resources.

## Setup

To install this module on your Puppet Master (or Puppet Client/Agent), use the
Puppet module installer:

    puppet module install google-gpubsub

Optionally you can install support to _all_ Google Cloud Platform products at
once by installing our "bundle" [`google-cloud`][bundle-forge] module:

    puppet module install google-cloud

Since this module depends on the `googleauth` and `google-api-client` gems,
you will also need to install those, with

    /opt/puppetlabs/puppet/bin/gem install googleauth google-api-client

If you prefer, you could also add the following to your puppet manifest:

		package { [
				'googleauth',
				'google-api-client',
			]:
				ensure   => present,
				provider => puppet_gem,
		}

## Usage

### Credentials

All Google Cloud Platform modules use an unified authentication mechanism,
provided by the [`google-gauth`][] module. Don't worry, it is automatically
installed when you install this module.

```puppet
gauth_credential { 'mycred':
  path     => $cred_path, # e.g. '/home/nelsonjr/my_account.json'
  provider => serviceaccount,
  scopes   => [
    'https://www.googleapis.com/auth/pubsub',
  ],
}
```

Please refer to the [`google-gauth`][] module for further requirements, i.e.
required gems.

### Examples

#### `gpubsub_topic`

```puppet
gpubsub_topic { 'conversation-1':
  ensure     => present,
  project    => $project, # e.g. 'my-test-project'
  credential => 'mycred',
}

```

#### `gpubsub_subscription`

```puppet
gpubsub_subscription { 'subscription-1':
  ensure               => present,
  topic                => 'conversation-1',
  push_config          => {
    push_endpoint => 'https://myapp.graphite.cloudnativeapp.com/webhook/sub1',
  },
  ack_deadline_seconds => 300,
  project              => $project, # e.g. 'my-test-project'
  credential           => 'mycred',
}

```


### Classes

#### Public classes

* [`gpubsub_topic`][]:
    A named resource to which messages are sent by publishers.
* [`gpubsub_subscription`][]:
    A named resource representing the stream of messages from a single,
    specific topic, to be delivered to the subscribing application.

### Parameters

#### `gpubsub_topic`

A named resource to which messages are sent by publishers.


#### Example

```puppet
gpubsub_topic { 'conversation-1':
  ensure     => present,
  project    => $project, # e.g. 'my-test-project'
  credential => 'mycred',
}

```

#### Reference

```puppet
gpubsub_topic { 'id-of-resource':
  name       => string,
  project    => string,
  credential => reference to gauth_credential,
}
```

##### `name`

  Name of the topic.


#### `gpubsub_subscription`

A named resource representing the stream of messages from a single,
specific topic, to be delivered to the subscribing application.


#### Example

```puppet
gpubsub_subscription { 'subscription-1':
  ensure               => present,
  topic                => 'conversation-1',
  push_config          => {
    push_endpoint => 'https://myapp.graphite.cloudnativeapp.com/webhook/sub1',
  },
  ack_deadline_seconds => 300,
  project              => $project, # e.g. 'my-test-project'
  credential           => 'mycred',
}

```

#### Reference

```puppet
gpubsub_subscription { 'id-of-resource':
  ack_deadline_seconds => integer,
  name                 => string,
  push_config          => {
    push_endpoint => string,
  },
  topic                => reference to gpubsub_topic,
  project              => string,
  credential           => reference to gauth_credential,
}
```

##### `name`

  Name of the subscription.

##### `topic`

  A reference to a Topic resource.

##### `push_config`

  If push delivery is used with this subscription, this field is used to
  configure it. An empty pushConfig signifies that the subscriber will
  pull and ack messages using API methods.

##### push_config/push_endpoint
  A URL locating the endpoint to which messages should be pushed.
  For example, a Webhook endpoint might use
  "https://example.com/push".

##### `ack_deadline_seconds`

  This value is the maximum time after a subscriber receives a message
  before the subscriber should acknowledge the message. After message
  delivery but before the ack deadline expires and before the message is
  acknowledged, it is an outstanding message and will not be delivered
  again during that time (on a best-effort basis).
  For pull subscriptions, this value is used as the initial value for
  the ack deadline. To override this value for a given message, call
  subscriptions.modifyAckDeadline with the corresponding ackId if using
  pull. The minimum custom deadline you can specify is 10 seconds. The
  maximum custom deadline you can specify is 600 seconds (10 minutes).
  If this parameter is 0, a default value of 10 seconds is used.
  For push delivery, this value is also used to set the request timeout
  for the call to the push endpoint.
  If the subscriber never acknowledges the message, the Pub/Sub system
  will eventually redeliver the message.



### Bolt Tasks


#### `tasks/publish.rb`

  Publish a message to a specific topic.

This task takes inputs as JSON from standard input.

##### Arguments

  - `topic`:
    The name of the topic to send the message to.

  - `attributes`:
    Optional attributes in { name => val } format (default: {})

  - `message`:
    The message to be published.

  - `project`:
    the project name where the cluster is hosted

  - `credential`:
    Path to a service account credentials file


## Limitations

This module has been tested on:

* RedHat 6, 7
* CentOS 6, 7
* Debian 7, 8
* Ubuntu 12.04, 14.04, 16.04, 16.10
* SLES 11-sp4, 12-sp2
* openSUSE 13
* Windows Server 2008 R2, 2012 R2, 2012 R2 Core, 2016 R2, 2016 R2 Core

Testing on other platforms has been minimal and cannot be guaranteed.

## Development

### Automatically Generated Files

Some files in this package are automatically generated by
[Magic Modules][magic-modules].

We use a code compiler to produce this module in order to avoid repetitive tasks
and improve code quality. This means all Google Cloud Platform Puppet modules
use the same underlying authentication, logic, test generation, style checks,
etc.

Learn more about the way to change autogenerated files by reading the
[CONTRIBUTING.md][] file.

### Contributing

Contributions to this library are always welcome and highly encouraged.

See [CONTRIBUTING.md][] for more information on how to get
started.

### Running tests

This project contains tests for [rspec][], [rspec-puppet][] and [rubocop][] to
verify functionality. For detailed information on using these tools, please see
their respective documentation.

#### Testing quickstart: Ruby > 2.0.0

```
gem install bundler
bundle install
bundle exec rspec
bundle exec rubocop
```

#### Debugging Tests

In case you need to debug tests in this module you can set the following
variables to increase verbose output:

Variable                | Side Effect
------------------------|---------------------------------------------------
`PUPPET_HTTP_VERBOSE=1` | Prints network access information by Puppet provier.
`PUPPET_HTTP_DEBUG=1`   | Prints the payload of network calls being made.
`GOOGLE_HTTP_VERBOSE=1` | Prints debug related to the network calls being made.
`GOOGLE_HTTP_DEBUG=1`   | Prints the payload of network calls being made.

During test runs (using [rspec][]) you can also set:

Variable                | Side Effect
------------------------|---------------------------------------------------
`RSPEC_DEBUG=1`         | Prints debug related to the tests being run.
`RSPEC_HTTP_VERBOSE=1`  | Prints network expectations and access.

[magic-modules]: https://github.com/GoogleCloudPlatform/magic-modules
[CONTRIBUTING.md]: CONTRIBUTING.md
[bundle-forge]: https://forge.puppet.com/google/cloud
[`google-gauth`]: https://github.com/GoogleCloudPlatform/puppet-google-auth
[rspec]: http://rspec.info/
[rspec-puppet]: http://rspec-puppet.com/
[rubocop]: https://rubocop.readthedocs.io/en/latest/
[`gpubsub_topic`]: #gpubsub_topic
[`gpubsub_subscription`]: #gpubsub_subscription

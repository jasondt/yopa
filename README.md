<pre>
_/\/\____/\/\____/\/\/\/\____/\/\/\/\/\________/\/\_____
_/\/\____/\/\__/\/\____/\/\__/\/\____/\/\____/\/\/\/\___
___/\/\/\/\____/\/\____/\/\__/\/\/\/\/\____/\/\____/\/\_
_____/\/\______/\/\____/\/\__/\/\__________/\/\/\/\/\/\_
_____/\/\________/\/\/\/\____/\/\__________/\/\____/\/\_
________________________________________________________
</pre>

_YOPA is Your Own Personal AWS_

It allows developers to code without being connected to the net.

[![Build Status](https://travis-ci.org/unbounce/yopa.svg)](https://travis-ci.org/unbounce/yopa)
<br/>
[![Clojars Project](http://clojars.org/com.unbounce/yopa/latest-version.svg)](http://clojars.org/com.unbounce/yopa)

Also available as a [Docker image](https://registry.hub.docker.com/u/unbounce/yopa/).

# Features

- SQS service, courtesy of https://github.com/adamw/elasticmq
- S3 service, courtesy of https://github.com/jubos/fake-s3
- SNS service with support for pre-defined (raw or wrapped) SQS, HTTP and HTTPS subscriptions.
  Note that the SNS service does not confirm HTTP/S subscriptions nor retry HTTP/S failed deliveries.
- EC2 Metadata service.


## Build and run

Bring the rubygems in with:

    mvn -f fake-s3-pom.xml clean initialize

Run the integration tests:

    lein test

Run with the example config:

    lein trampoline run --config yopa-config-example.yml


## Usage

Run `lein run` to get help.

The following examples assume all the default configuration parameters are used.


### AWS CLI

    aws --endpoint-url http://localhost:47196 sns list-topics


### Ruby

    AWS.config(:access_key_id => 'x',
               :secret_access_key => 'x',
               :region => 'yopa-local',
               :use_ssl => false,
               :sqs_endpoint => 'localhost',
               :sqs_port => 47195,
               :sns_endpoint => 'localhost',
               :sns_port => 47196)


### JVM

You can either set the endpoints explicitly, like with:

    AmazonSNS snsClient = new AmazonSNSClient(awsCredentials);
    snsClient.setEndpoint("http://localhost:47196");

Or you can configure the SDK to load the regions override configuration generated by Yopa and get the `yopa-local` region:

    System.setProperty(SDKGlobalConfiguration.REGIONS_FILE_OVERRIDE_SYSTEM_PROPERTY, "/tmp/aws-regions-override.xml");
    Region region = RegionUtils.getRegion("yopa-local");

and then either set the region on the client, while forcing the client to use HTTP:

    AmazonSNS snsClient = new AmazonSNSClient(awsCredentials, new ClientConfiguration().withProtocol(Protocol.HTTP));
    snsClient.setRegion(region);

or derive the endpoint from the region:

    AmazonSNS snsClient = new AmazonSNSClient(awsCredentials);
    snsClient.setEndpoint("http://" + region.getServiceEndpoint("sns"));

To use the EC2 Metadata service, set the SDK overriding property to point to Yopa's SNS port, using the `/ec2-metadata` path:

    System.setProperty(SDKGlobalConfiguration.EC2_METADATA_SERVICE_OVERRIDE_SYSTEM_PROPERTY, "http://0.0.0.0:47196/ec2-metadata");


### Accessing S3

Using Yopa's S3 service requires setting the S3 client to use the `forced path style`.
Check out [this detailed clients guide](https://github.com/jubos/fake-s3/wiki/Supported-Clients)
to see how to do it for your platform.

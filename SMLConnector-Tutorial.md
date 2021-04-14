# Learn CSV File

This tutorial will show you how to learn a CSV file through the SML Kafka Connector.

We will use Confluent's Kafka Connect SFTP Source Connector as the source connector that loads the CSV file into a Kafka topic. The version of it that we use in this tutorial is 2.2.4.


## Start a Kafka Cluster
* For example, start a local Kafka Docker Cluster through cp-all-in-one at https://github.com/confluentinc/cp-all-in-one with `docker-compose up -d`. Let the cluster run just one container per service.


## Deploy SymetryML to an accessible Jetty server
* For example, deploy SymetryML to a Jetty server on a CentOS VM in a MacBook.


## Copy the SML Kafka Connector package to Kafka Connect
* Unzip symetryml-kafka-connector-1.0.3.zip locally:  
	`docker cp symetryml-kafka-connector-1.0.3 connect:/usr/share/java/`  
* We assume that /usr/share/java is in CONNECT_PLUGIN_PATH.
* Ensure that the unix file permissions are sufficient. 755 should do.


## Copy the Kafka Connect SFTP Source Connector package to Kafka Connect
* Unzip confluentinc-kafka-connect-sftp-2.2.4.zip locally.  
 	`docker cp confluentinc-kafka-connect-sftp-2.2.4 connect:/usr/share/java/`  
* See https://docs.confluent.io/kafka-connect-sftp/current/index.html.
* Find the link to download the zip file there.
* Ensure that the unix file permissions are sufficient. 755 should do.


## Deploy the Connectors
For example, for a Kafka cluster:  
	`docker restart connect`


## Populate the Kafka Topic with the CSV file

* Configure etc/sftp_housing_source.json.
* Set `input.path` to the "etc" directory, which contains housing.csv.
* Set `error.path`, "finished.path" to the "temp" directory.
* Set `sftp.username`, `sftp.password`, "sftp.host" as required.  
* On Mac, enable Remote Login in System Preferences/Sharing in order to enable the SFTP server. See [https://www.maciverse.com/how-to-turn-on-your-macs-sftp.html].
* From the ${SMLCONNECTORZIP} folder:
`curl -XPOST -H "Content-Type: application/json" --data @etc/sftp_housing_source.json http://localhost:8083/connectors`


## Learn from the Kafka Topic

* From the ${SMLCONNECTORZIP} folder:
* Configure etc/quickstart-symetryml-properties.json.
* Set `sml.url`, `sml.customer.id`, `sml.customer.secret.key` as required.
* For example, `sml.url`: `http://centosvm:8080/symetry/rest`
* On Mac, use `http://host.docker.internal:8081` instead of `http://localhost:8081` for `value.converter.schema.registry.url`.
* Start the SML connector:
`curl -XPOST -H "Content-Type: application/json" --data @etc/quickstart-symetryml-properties.json http://localhost:8083/connectors`
* Explore the project specified by "sml.project" in order to verify that it learned "housing.csv" from the topic "sml_housing".


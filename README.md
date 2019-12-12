# Packet Labs IoT workshop

This workshop deploys compute, storage, networking, and an IoT application to Packet.com.

## Conceptual architecture

Diagram:

![Conceptual architecture](/docs/images/conceptual.png)

Component parts:

* Kubernetes - provisioned with [Terraform](https://www.terraform.io)
* TLS termination - via [cert-manager](https://cert-manager.io)
* MQTT Broker - [emitter.io](https://emitter.io)
* Compute platform - [OpenFaaS](https://github.com/openfaas/faas)
* MQTT Connector - [openfaas-incubator/mqtt-connector](https://github.com/openfaas-incubator/mqtt-connector)
* Database/storage - [Postgresql](https://www.postgresql.org)
* Docker registry - deployed externally, i.e. the Docker Hub.
* Ingress Controller - Traefik or Nginx, depending on preference

## Getting started

Everything you need to deploy this workshop is available in this repository.

### 1) Clone the repo

```sh
git clone https://github.com/packet-labs/iot
```

### 2) Create a bare-metal Kubernetes cluster

You will need to [install Terraform](https://www.terraform.io) for this step.

* Set your Packet API and project ID in the .tf files in `/k8s`.

* Enter [the `k8s` folder](/k8s/) and apply the terraform plan.

### 3) Install Postgres via KubeDB and helm

You will need to install helm for this step.

* Install [postgresql](/postgresql/)

### 4) Add the MQTT Broker (Emitter.io)

* Install [emitter](/emitter/)

More info at: https://emitter.io

* Navigate to `http://IP:port/keygen` where `IP` is the address of your Emitter.io deployment

* Generate a channel key for the topic `drone-position`

![](/docs/images/keygen.png)

You'll need this key to configure the OpenFaaS Connector

### 5) Install OpenFaaS

* Install [openfaas](/openfaas/) to provide compute and events

* Deploy the [OpenFaaS services](/openfaas/services/) for the application

You will also deploy the [schema.sql](/openfaas/services/schema.sql) at this time for `drone_position` and `drone_event`.

* [Deploy TLS with cert-manager (optional)](https://blog.alexellis.io/tls-the-easy-way-with-openfaas-and-k3sup/)

### 6) Add the OpenFaaS MQTT-Connector

The MQTT-Connector is used to trigger functions and services in response to messages generated by the event source.

The topic is `drone-position`.

```sh
git clone https://github.com/openfaas-incubator/mqtt-connector
cd mqtt-connector
```

Edit `chart/mqtt-connector/values.yaml` and update the CHANNEL_KEY value from the step above.

Apply the changes:

```sh
cd mqtt-connector/chart/

helm template -n openfaas --namespace openfaas mqtt-connector/ --values mqtt
-connector/values.yaml | kubectl apply -f -
```

### 7) Add Grafana for function/service visualization

Grafana packages a pre-compiled dashboard for OpenFaaS to show metrics like throughput and latency.

* [Deploy and Grafana](/grafana/)

## Todo/backlog

- [ ] Select deploy business insights software (Metabase / Redash?) - Alan & team
- [ ] Write deployment instructions for Metabase or Redash - Alan & team
- [ ] Create Python or Node app to publish generated MQTT data - Alan & team
- [x] Create Postgesql schema asset for drone geo-location data - Alex
- [x] Create Postgesql schema asset for drone event data that can support the following events (jsonb or string of JSON) - Alex

* Collision Avoidance
* Airspace Violation
* Device Health Report
* Low Battery
* Excessive Battery Usage
* Device Fault

- [x] Create OpenFaaS Function: insert geo row (using `node12` template) - Alex
- [ ] Create OpenFaaS Function: insert event row (using `node12` template) - Alan & team 

- [ ] Create OpenFaaS Function: process geo data for collision or airspace (using `node12` template) - Alan & team

This should detect, log the event, and emit a message back thru mqtt to the violating drones

- [ ] Create OpenFaaS Function: Query drone positions for mapbox (using `node12` template)
- [ ] Create OpenFaaS Function: Query drone events for mapbox (using `node12` template)
- [ ] Create OpenFaaS service for viewing data (mapbox for geo) (using `node12` template) - Alan & team

* Static example with OpenStreetMap https://github.com/alexellis/sf-assets
* Pure express - https://github.com/openfaas-incubator/node10-express-service
* Mix of function/express - https://github.com/openfaas/openfaas-cloud/blob/master/dashboard/of-cloud-dashboard/handler.js
* Static example - https://www.openfaas.com/blog/serverless-static-sites/

Suggested schema ('drone_status'):

* ID 
* Name
* Altitude 
* GPS Latitude
* GPS Longitude
* Velocity (sent as a vector)
* Battery
* Temperature

Suggested schema ('drone_event'):

* ID 
* Event Type
* Data 

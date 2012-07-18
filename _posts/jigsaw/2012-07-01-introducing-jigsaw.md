---
layout: post
category : Jigsaw
tagline: Making ITK Work for you
tags : [digital signature, itk, soap, tutorial]
---
{% include JB/setup %}

[Jigsaw](https://github.com/devmikey/Jigsaw) is an implementation of the Interoperability Toolkit (ITK) specifications for the NHS.



It has been written in javascript and runs on node.js and uses mongoDB as the data store.

## Features

Jigsaw is currently under active development but currently supports the following:

### ITK Client

* Creation of Soap packets
* Creation of Digital Signatures
* Creation of the Distribution Envelope

### ITK Middleware

* Supports the Request/Exception Message Interaction
* Supports the Request/Response/Exception Interaction Pattern
* Supports the Queue Collection Service

## Usage

At the moment Jigsaw provides a basic ITK client and Host, it is envisioned that Jigsaw will be used in the following ways:

* Development - used to assist developers building ITK compliant components
* Testing - used by testing teams to build independent test cases for an ITK component
* Client production - where the client code is used to access ITK Compliant Middleware in production
* Host production -  where the host is used to provide ITK compliant access to an organisation services in production

The host can be configured to accept different messages and endpoints.

The client can generates digitally signed messages and supports the following Interaction Patterns:


An example Host :

	var jigsaw = require('jigsaw')
	var interactionHandler = jigsaw.interactionHandler()
	
	var documentProcessor = function(req,res){
		console.log(req)
	}
	
	var routes = new Array();
	routes.push(interactionHandler.create("/sync/clinicaldocuments", "requestException", [], documentProcessor))
	var app = jigsaw.createServer(routes)
	app.addKey("client_public.pem")
	app.listen(3000)
	
An example client :

	var client = require('jigsaw/client/client');
	var distributionEnvelope = require('jigsaw/client/messages/distributionEnvelope');

	var msgProperties = {
		"serviceName" : "urn:nhs-itk:services:201005:SendDocument-v1-0",
		"key":  "client.pem",
		"properties": 
			{ "addresslist": new Array("urn:nhs-uk:addressing:ods:Y88764:G1234567"),
				"auditIdentity": new Array("urn:nhs-uk:addressing:ods:R59:oncology"),
				"manifest": [
					{"mimetype": "application/pdf", "data": "base64 encode file 1"},
					{"mimetype": "application/pdf", "data": "base64 encode file 2"}
				],
				"senderAddress": "urn:nhs-uk:addressing:ods:R59:oncology"
			},
		"url": "http://localhost:3000/sync/clinicaldocuments",
		"handler": function(ctx) {
			console.log("http response code:" + ctx.statusCode);
			console.log("soap:" + ctx.response);
		} 
	}
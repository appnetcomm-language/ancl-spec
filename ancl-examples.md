# Introduction

This demonstrates some examples of ANCL models, and some ANCL models for common patterns.

# Web Service

Simple 2-tier (app+db) Web Service. "prod" and "dev" environments.

    models:
    - name: mywebsvc
      components:
      - name: user
        ingress: {}
        egress:
        - web::https
      - name: web
        ingress:
        - name: https
          ports:
          - [443,443,"tcp"]
        egress:
        - db::sql
      - name: db
        ingress:
        - name: sql
          ports:
          - [3306,3306,"tcp"]

    contexts:
    - name: prod
    - name: dev

    roles:
    - name: "0.0.0.0/0"
      roles: [ prod::mywebsvc::client ]
    - name: "192.0.2.16/32"
      metadata:
        dnsname: "web01.prod.example.com"
      roles: [ prod::mywebsvc::web ]
    - name: "192.0.2.32/32"
      metadata:
        dnsname: "db01.prod.example.com"
      roles: [ prod::mywebsvc::db ]
    - name: "192.0.2.224/32"
      metadata:
        dnsname: "web01.dev.example.com"
      roles: [ dev::mywebsvc::web ]
    - name: "192.0.2.225/32"
      metadata:
        dnsname: "db01.dev.example.com"
      roles: [ dev::mywebsvc::db ]
    - name: "192.0.2.254/32"
      metadata:
        dnsname: "testclient.dev.example.com"
      roles: [ dev::mywebsvc::client ]

# Load Balancer with Different Frontside/Backside Addresses

While many proxy based load balancers run with a single IP address for both the incoming traffic from the clients and the outgoing traffic to the backends, some are able to separate those out into different IP addresses and interfaces. This show how one might realize that in ANCL.

    models:
    - name: AppLB
      components:
      - name: client
        egress:
        - lbfront::listener
        ingress: []
      - name: lbfront
        egress: []
        ingress:
        - name: "listener"
          ports:
          - [80,80,"tcp"]
      - name: lbback
        egress:
          - name: backend::http
        ingress: []
      - name: backend
        egress: []
        ingress:
          - name: "http"
            ports:
            - [80,80,"tcp"]

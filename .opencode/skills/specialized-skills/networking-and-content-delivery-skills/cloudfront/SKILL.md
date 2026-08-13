---
name: cloudfront
description: >
  Configures Amazon CloudFront content delivery across six workflows: when to use CloudFront and
  how it fits with AWS WAF, Shield, CloudFront Functions, Lambda@Edge, Route 53, and origins
  (creating a distribution, caching, and Flat Rate Pricing (FRP) versus pay-as-you-go pricing); managing
  custom-domain TLS certificates (ACM in us-east-1); configuring multi-tenant distributions;
  protecting origins with origin access control (OAC), VPC origins, and origin mutual TLS (mTLS);
  securing content with signed URLs and cookies, geographic restrictions, viewer mutual TLS, and
  edge token validation; and observing traffic with standard and real-time logs. Applicable when the
  customer wants to put CloudFront in front of content, choose pricing, lock an origin, restrict who
  can view content, or analyze logs. Not applicable for the Route 53 DNS side of a CloudFront custom
  domain or failover between distributions (see the route53-cloudfront skill), or for pure-Route 53
  DNS work (see the route53 skill).
---
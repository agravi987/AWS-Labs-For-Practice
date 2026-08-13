---
name: waf
description: >-
  Configures AWS WAF to filter web traffic: creating web access control lists (web ACLs) on
  CloudFront, Application Load Balancers, API Gateway, and AppSync; AWS Managed Rules tuned in Count
  mode; rate-based rules for HTTP floods; IP set and geographic match rules; Bot Control (Common and
  Targeted); turning bot labels into a confidence signal; stripping spoofed inbound x-amzn-waf-*
  headers; recovering the real client IP behind a CDN; Fraud Control (account takeover and account
  creation fraud prevention); and logging and request sampling. Use when the user wants to protect a
  web application or API from common exploits, bots, credential stuffing, fake-account creation, or
  HTTP floods at the application layer (layer 7). Routes to the right per-task procedure in
  references. Do NOT use for L3/L4 DDoS protection (shieldadvanced skill), multi-account WAF rollout
  (firewallmanager skill), CloudFront configuration (cloudfront skill), or Route 53 health checks or
  records (route53 skill).
---
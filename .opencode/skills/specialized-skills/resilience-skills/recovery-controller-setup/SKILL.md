---
name: recovery-controller-setup
description: >
  Configures AWS Application Recovery Controller (ARC) for operational resilience:
  routing controls with safety rules for cross-Region failover, and zonal shift / zonal autoshift
  for AZ-impairment recovery. Applies when setting up failover routing, configuring safety rules,
  enabling zonal shift, or configuring zonal autoshift with practice runs. Also applies when
  shifting traffic out of a specific Availability Zone (AZ) for an ALB/NLB or other resource.
  For a broader "an AZ is impaired, what is my response across services" question, see
  aws-resilience-lifecycle. Does not apply to Resilience Hub setup or FIS experiments.
---
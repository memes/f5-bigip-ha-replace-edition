# Replace F5 BIG-IP edition in an HA pair

![GitHub release](https://img.shields.io/github/v/release/memes/repo-template?sort=semver)
![Maintenance](https://img.shields.io/maintenance/yes/2023)
[![Contributor Covenant](https://img.shields.io/badge/Contributor%20Covenant-2.1-4baaaa.svg)](CODE_OF_CONDUCT.md)

This repo is a guide to changing the BIG-IP edition of a production HA pair in
Google Cloud with minimal disruption to traffic and preserving your infrastructure-as-code
state so it matches actual infrastructure resources for each stage of the process.
For example, you can upgrade from Good to Best+ edition, or transition VEs from
PAYG to perpetual licenses (BYOL), or vice-versa.

While the examples are specific to GCP the general technique should be equally
applicable to other private and public cloud deployments.

> NOTE: These techniques cannot be used to **change** the BIG-IP version of the
> deployment - *ConfigSync* continues to manage cluster state and thus requires
> that the base BIG-IP versions are identical for all devices in the group.

## Scenarios in this repo

|Initial Solution|Upstream version|Scenario|Notes|
|-----------------|----------------|-------|-----|
|[GDM v1 via-API, 3nic, PAYG]|v4.3.0|Throughput change(25Mbps -> 1Gbps)||

## Approach taken

All the scenarios follow the basic steps:

0. Starting point

   ![Initial group](images/via-api/Initial%20group.png)

   * *BIG-IP VE 1* and *BIG-IP VE 2* are an existing HA pair deployed using one of
     F5's supported or community repositories
   * The cluster is in a known-good state

1. Expand the cluster; add new VEs running target edition

   > NOTE: This step is specific to your starting point; in order to preserve
   > IaC state and avoid future issues follow the directions in **step-1-expand**
   > folder for your scenario.

   ![Expand](images/via-api/Expanded%20group.png)

   * Two new BIG-IP VEs are added to GCP; *BIG-IP VE 3* and *BIG-IP VE 4*
   * *BIG-IP VE 3* and *BIG-IP VE 4* are functionally standalone
   * All four VEs should be running the same set of extensions (e.g. DO, AS3,
     CFE - if applicable, etc.) and the same release versions of the extensions

2. Extend BIG-IP HA configuration to new VEs, while removing old VEs from configuration

   Good news! These steps are common to all the examples and could serve as a
   starting point if your specific scenario is not in this repo. A detailed
   walkthrough can be found in [Modifying HA].

   a. Extend *device trust* to include the new VEs

      ![Expand](images/via-api/Added%20to%20device%20trust%20group.png)

   b. **Force Offline** the new VEs

      ![Forced offline](images/via-api/Forced%20offline.png)

   c. Extend the *failover group* to the new VEs

      ![Expand](images/via-api/Added%20to%20failover%20group.png)

   d. Release Offline and force failover to a new VE

      ![Expand](images/via-api/Force%20to%20standby.png)

   e. Reduce failover group to exclude the original VEs

      ![Expand](images/via-api/Reduce%20failover%20group.png)

   f. Reduce device trust to exclude the original VEs

      ![Expand](images/via-api/Reduce%20device%20trust.png)

3. Remove the original VEs from deployment

   > NOTE: This step is specific to your starting point; in order to preserve
   > IaC state and avoid future issues follow the directions in **step-3-reduce**
   > folder for your scenario.

   ![Expand](images/via-api/Final%20group.png)

[GDM v1 via-API, 3nic, PAYG]: https://github.com/F5Networks/f5-google-gdm-templates/tree/v4.3.0/supported/failover/same-net/via-api/3nic/existing-stack/payg
[Modifying HA]: README.md

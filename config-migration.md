


# DEVEAST upgrade steps
March 27th, 2017

> ERRATA: Pushing config to origin may require the --force switch due to a bug in the crc check.

## AWS prep

Nuke all ec2's except NAT and api-router.


## Config prep

Add dg-superkaiju tier, deployables, product and tenant:

```bash
dconf tier add DEVEAST --is-dev
dconf deployable register all --tier DEVEAST
dconf organization add directivegames dg
dconf product add dg-superkaiju
dconf tenant add dg-superkaiju-deveast dg-superkaiju

# call drift-admin deployable register. it's a work in progress...
drift-admin --tier DEVEAST --config dgnorth deployable register

driftconfig addtenant dgnorth -n dg-superkaiju-deveast -t DEVEAST -o directivegames -p dg-superkaiju -d drift-base
driftconfig push dgnorth
```




The config is missing the auto-filled entries for aws. This needs to be added manually until the provision logic is fully rigged.

domain.json:

```json
    "aws": {
        "ami_baking_region": "eu-west-1"
    ...
```

tiers.json:

```json
    {
        "tier_name": "DEVEAST",
        "aws": 
        {
            "region": "ap-southeast-1",
            "ssh_key": "livenorth-key"            
        }, 
        "celery_broker_url": "redis://redis.deveast.dg-api.com:6379/15",
        "service_user": 
        {
      
       "resource_defaults": [
      
      
        ...
    }
    
```

authentication/public-keys.json:

```
...

    {
    ...
    
    DUPLICATE ENTRY ABOVE FOR superkaiju-backend AS WELL!
```

deployables.json:

```json

    {

```

> Also had to make a new file in legacy auth config. See **directive-tiers/tiers/DEVEAST/superkaiju-backend.json** for more info.

## DNS entry for the Endpoint
Make a DNS entry for the tenant:

```
# In Route53 AWS console:
Name:  dg-superkaiju-deveast.dg-api.com
Type:  A record
Alias: dualstack.deveast-api-router-lb-auto2-625491317.ap-southeast-1.elb.amazonaws.com.
```

## Provision tenant resources

Run this on your local machine:

```bash
drift-admin --tier DEVEAST tenant dg-superkaiju-deveast create
```

## Prepare an AMI for drift-base

In the **drift-base** folder:

```bash
drift-admin ami bake
drift-admin --tier DEVEAST ami run
```


## Fixes made to superkaiju-backend code

#### superkaiju-backend project
All the files in drift-base/config copied over, renamed accordingly and the contents search-replaced for "drift-base" -> "superkaiju-backend".

> Remember to copy and fix up ALL THE FILES!

The project was baked and run:

```bash
# Executed from superkaiju-backend root folder:
drift-admin --tier DEVEAST ami bake
drift-admin --tier DEVEAST ami run
```

#### api-router
The current api-router is using the old config mgmt. The json file in directive-tiers/tiers/DEVEAST.JSON was updated manually with this, and then published:

```json
        {
            "name": "superkaiju-backend",
            "api": "superkaiju",
            "ssh_key": "deveast-key.pem"
        },
```


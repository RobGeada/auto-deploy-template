# auto-deploy-template

Template to demonstrate usage of auto-deploy. 

### Usage
Run `./auto-deploy -f` will auto-deploy based on the configuration specified in auto-deploy.config
Run `./auto-deploy -c [CLUSTER_NAME] -u [CLUSTER_USERNAME] -p [PROJECT_NAME]` to specify your own configuration. This new configuration will be saved in auto-deploy.config accordingly, allowing you to deploy with the same settings with `./auto-deploy -f`

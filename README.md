# auto-deploy

### Overview
With auto-deploy, Jupyter notebook can be deployed with their data onto Spark clusters in OpenShift without need to write Dockerfiles, manually create pods, or any of the other hassles that go with it. 

### Setup
Simply put all of your desired notebooks into a folder named `scripts`, your data into a folder named `data`, and then clone auto-deploy into the parent directory containing `scripts` and `data`. See (this template)[link] for an example!

### Usage
Run `./auto-deploy -c [OPENSHIFT CLUSTER] -u [OS USERNAME] -p [OS PROJECT]` to get started. Check out `./auto-deploy -h` for more advanced options. 

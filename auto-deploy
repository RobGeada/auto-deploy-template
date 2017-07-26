#!/bin/bash

#========================================================
#echo setup
#========================================================
NC="\x1b[39;49;00m"
GR='\033[0;32m'
RE='\033[0;31m'
DONE_MSG="${GR}[DONE]${NC}" 
FAIL_MSG="${RE}[FAIL]${NC}"

#========================================================
#log setup
#========================================================
LOG_DIR=$(pwd)/auto-deploy.log
echo "Deploy started at $(date)" > ${LOG_DIR}

#========================================================
#parse arguments
#========================================================
while getopts :o:u:c:p:hdsf opt; do
    case $opt in
        c)
            OS_CLUSTER=$OPTARG
            ;;
        u)
            OS_USER=$OPTARG
            USER_REQUESTED=true
            ;;
        d)  
            CLEAN=true
            ;;
        p)
            PROJECT=$OPTARG
            ;;
        s)
            SKIP=true
            ;;
        f) 
            FROM_CONFIG=true
            ;;
        h)
			echo
            echo "Usage: auto-deploy.sh [options]"
            echo
            echo "Automatic deployment of the Oshinko suite, notebook, and data into OpenShift "
            echo
            echo "optional arguments:"
            echo "  -h             show this help message"
            echo "  -c CLUSTER     OpenShift cluster url to login against (default: https://localhost:8443)"
            echo "  -u USER        OpenShift user to run commands as (default: $DEFAULT_OPENSHIFT_USER)"
            echo "  -p PROJECT     OpenShift project name (default: $DEFAULT_OPENSHIFT_PROJECT)"
            echo "  -s             Skip directory formatting and deploy"
            echo "  -f             Get parameters from config file"
            echo "  -d             Delete all auto-deploy created files"
            echo
            exit
            ;;
        \?)
            echo "Invalid option: -$OPTARG" >&2
            exit
            ;;
    esac
done

#========================================================
#load params from config, skip commands
#========================================================

if [ "$FROM_CONFIG" = true ]
then
    #make sure a config file exists
    if [ -d "$(pwd)/auto-deploy.config" ] ; 
    then
        echo "ERROR: No config file found!"
        exit
    fi

    #parse config file
    OS_CLUSTER=$(cat auto-deploy.config | grep "CLUSTER" | awk '{print $3}') 
    OS_USER=$(cat auto-deploy.config | grep "USER" | awk '{print $3}') 
    PROJECT=$(cat auto-deploy.config | grep "OC_PROJECT" | awk '{print $3}') 
fi

#========================================================
#deal with setting default parameters
#========================================================
function set_default {
    if [ -z $"$1" ]
    then
        echo "$2"
    else
        echo "$1"
    fi
}

OS_CLUSTER=$(set_default "$OS_CLUSTER" https://localhost:8443)
OS_USER=$(set_default "$OS_USER" developer)
PROJECT=$(set_default "$PROJECT" auto-deploy)
SKIP=$(set_default "$SKIP" false)
CLEAN=$(set_default "$CLEAN" false)
FROM_CONFIG=$(set_default "$FROM_CONFIG" false)

#========================================================
#check to make sure directory isn't already formmatted
#========================================================
if [ "$CLEAN" = false ] && [ "$SKIP" = false ] && [ -d "$(pwd)/notebook-template/" ]
then
    echo
    echo "ERROR: notebook-template already exists!"
    echo "Either remove notebook-template with ./auto-deploy -d or use ./auto-deploy -s to skip formatting!"
    echo 
    exit
fi

#========================================================
#output current run parameters to stdout and logs
#========================================================

echo | tee -a ${LOG_DIR}
echo "===AUTO-DEPLOY PARAMETERS==="
echo "OpenShift cluster:  ${OS_CLUSTER}" | tee -a ${LOG_DIR}
echo "OpenShift username: ${OS_USER}" | tee -a ${LOG_DIR}
echo "OpenShift project:  ${PROJECT}" | tee -a ${LOG_DIR}
echo | tee -a ${LOG_DIR}

#========================================================
#save params from config
#========================================================
echo CLUSTER = $OS_CLUSTER > auto-deploy.config
echo USER = $OS_USER >> auto-deploy.config
echo OC_PROJECT = $PROJECT >> auto-deploy.config

#========================================================
#code to skip formatting
#========================================================
if [ "$SKIP" = true ]
then
    cd notebook-template
    ./deploy.sh -c ${OS_CLUSTER} -u ${OS_USER} -p ${PROJECT}
    cat $(pwd)/deploy.log >> ${LOG_DIR}
    exit
fi

#========================================================
#directory cleaning code
#========================================================
if [ "$CLEAN" = true ]
then
    echo -n "Cleaning up generated files...         " 
    rm -Rf $(pwd)/notebook-template
    echo -e ${DONE_MSG}
fi

#========================================================
#here's the actual script
#========================================================

#get general directory structure into place
echo -n "Cloning format repo...                 " 
git clone https://github.com/RobGeada/notebook-template --quiet >> ${LOG_DIR}
echo -e "${DONE_MSG}"

#link data into right places
echo -n "Copying data into Docker contexts...   " 
cp -r $(pwd)/data $(pwd)/notebook-template/notebook/data >> ${LOG_DIR}
cp -r $(pwd)/data $(pwd)/notebook-template/spark-worker/data >> ${LOG_DIR}
echo -e "${DONE_MSG}"

#link scripts into right places
echo -n "Copying scripts into Docker contexts..." 
cp -r $(pwd)/scripts $(pwd)/notebook-template/notebook/scripts >> ${LOG_DIR}
echo -e "${DONE_MSG}"

#run deploy script
cd notebook-template
./deploy.sh -c ${OS_CLUSTER} -u ${OS_USER} -p ${PROJECT}

#grab deploy logs
cat $(pwd)/deploy.log >> ${LOG_DIR}
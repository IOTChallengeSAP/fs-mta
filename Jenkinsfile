#!groovy

import com.sap.piper.Utils
import com.sap.piper.ConfigurationLoader
import com.sap.piper.ConfigurationMerger

/**
 *	Copyright (c) 2017 SAP SE or an SAP affiliate company.  All rights reserved.
 * s
 *	This software is the confidential and proprietary information of SAP
 *	("Confidential Information"). You shall not disclose such Confidential
 *	Information and shall use it only in accordance with the terms of the
 *	license agreement you entered into with SAP.
*/

@Library('piper-lib-os') _

CONFIG_FILE_PROPERTIES = 'x.pipeline/config.properties'
CONFIG_FILE_YML = 'x.pipeline/config.yml'

node() {
  //Global variables:
  APP_PATH = 'src'
  SRC = "${WORKSPACE}/${APP_PATH}"

  def CONFIG_FILE

  def STEP_CONFIG_NEO_DEPLOY='neoDeploy'
  def STEP_CONFIG_MTA_BUILD='mtaBuild'

  stage("Clone sources and setup environment"){
    deleteDir()
    Map neoDeployConfiguration, mtaBuildConfiguration
    dir(APP_PATH) {
      checkout scm
      if(fileExists(CONFIG_FILE_YML) ) {
          CONFIG_FILE = CONFIG_FILE_YML
      } else if(fileExists (CONFIG_FILE_PROPERTIES) ) {
          CONFIG_FILE = CONFIG_FILE_PROPERTIES
      } else {
          error "No config file found."
      }
      echo "[INFO] using configuration file '${CONFIG_FILE}'."
      setupCommonPipelineEnvironment script: this, configFile: CONFIG_FILE
      prepareDefaultValues script: this
      neoDeployConfiguration = ConfigurationMerger.merge([:], (Set)[],
                                                         ConfigurationLoader.stepConfiguration(this, STEP_CONFIG_NEO_DEPLOY), (Set)['neoHome', 'account'],
                                                         ConfigurationLoader.defaultStepConfiguration(this, 'neoDeploy'))
      mtaBuildConfiguration = ConfigurationMerger.merge([:], (Set)[],
                                                        ConfigurationLoader.stepConfiguration(this, STEP_CONFIG_MTA_BUILD), (Set)['mtaJarLocation'],
                                                        ConfigurationLoader.defaultStepConfiguration(this, 'mtaBuild'))
    }
    echo "VOR MTA_BUILD"
    MTA_JAR_LOCATION = commonPipelineEnvironment.getConfigProperty('MTA_HOME')
    echo "MTA Jar Location: ${MTA_JAR_LOCATION}"
    NEO_HOME = neoDeployConfiguration.neoHome ?: commonPipelineEnvironment.getConfigProperty('NEO_HOME')
    proxy = commonPipelineEnvironment.getConfigProperty('proxy') ?: ''
    httpsProxy = commonPipelineEnvironment.getConfigProperty('httpsProxy') ?: ''
  }

  stage("Build Fiori App"){
    dir(SRC){
      withEnv(["http_proxy=${proxy}", "https_proxy=${httpsProxy}"]) {
        MTAR_FILE_PATH = mtaBuild script: this, mtaJarLocation: MTA_JAR_LOCATION, buildTarget: 'CF', dockerImage: 'felixgehtlos/scp-deployment'
      }
    }
  }
  
  stage("Deploy Fiori App"){
  echo "hier im Deploy"
    dir(SRC){
      withEnv(["http_proxy=${proxy}", "https_proxy=${httpsProxy}"]) {
        neoDeploy script: this, neoHome: NEO_HOME, archivePath: MTAR_FILE_PATH
      }
    }
  }
}
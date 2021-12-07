node{
    def SERVER_KEY_CREDENTIALS_ID=env.SERVER_KEY_CREDENTIALS_ID
    def DEPLOYDIR = 'toDeploy'
    def APIVERSION = '51.0'
    def toolbelt = tool 'toolbelt'
	def SF_INSTANCE_URL = env.SF_INSTANCE_URL
	def SF_CONSUMER_KEY = env.SF_CONSUMER_KEY
	def SF_USERNAME = env.SF_USERNAME
	def DELTACHANGES = 'deltachanges'
	
	

    if ((params.PreviousCommitId == '') || (params.LatestCommitId == ''))
	{
		error("Please enter both Previous and Latest commit IDs")
	}
    if (params.PreviousCommitId == params.LatestCommitId)
	{
		error("Previous and Latest Commit IDs can't be same.")
	}
    if (TESTLEVEL=='RunSpecifiedTests')
	{
		if (params.SpecifyTestClass == '')
		{
			error("Please Specify Test classes")
		}
	}
    stage('Clean Workspace') {
        try {
            deleteDir()
        }
        catch (Exception e) {
            println('Unable to Clean WorkSpace.')
        }
    }
    stage('checkout source') {
        checkout scm
    }
    withEnv(["HOME=${env.WORKSPACE}"]) {	
	
	    withCredentials([file(credentialsId: SERVER_KEY_CREDENTIALS_ID, variable: 'server_key_file')]) {

		stage('Authorize to Salesforce') {
			rc = command "${toolbelt}/sfdx auth:jwt:grant --instanceurl ${SF_INSTANCE_URL} --clientid ${SF_CONSUMER_KEY} --jwtkeyfile ${server_key_file} --username ${SF_USERNAME} --setalias ${SF_USERNAME}"
		    if (rc != 0) {
			error 'Salesforce org authorization failed.'
		    }
		}
        stage('Delta changes')
		{
			script
            {
				//bat "echo y | sfdx plugins:install sfpowerkit"
				rc = command "${toolbelt}/sfdx sfpowerkit:project:diff --revisionfrom %PreviousCommitId% --revisionto %LatestCommitId% --output ${DELTACHANGES} --apiversion ${APIVERSION} -x"
            }
        }
		stage('Convert metadeta')
		{
			script
			{
				dir("DeltaChanges")
				{
					rc = command "${toolbelt}/sfdx force:source:convert -d ../toDeploy"
				}
			}
		}
        stage('Validate Only') 
		{
			if (Deployment_Type=='Validate Only')
			{
				script
				{
				
					if (TESTLEVEL=='NoTestRun') 
					{
						println TESTLEVEL
						rc = command "${toolbelt}/sfdx force:mdapi:deploy -d ${DEPLOYDIR} --checkonly --wait 10 --targetusername ${SF_USERNAME} "
					}
					else if (TESTLEVEL=='RunLocalTests') 
					{
						println TESTLEVEL
						rc = command "${toolbelt}/sfdx force:mdapi:deploy -d ${DEPLOYDIR} --checkonly --wait 10 --targetusername ${SF_USERNAME} --testlevel ${TESTLEVEL} --verbose --loglevel fatal"
					}
					else if (TESTLEVEL=='RunSpecifiedTests')
					{
						println TESTLEVEL
						def Testclass = SpecifyTestClass.replaceAll('\\s','')
						println Testclass
						rc = command "${toolbelt}/sfdx force:mdapi:deploy -d ${DEPLOYDIR} --checkonly --wait 10 --targetusername ${SF_USERNAME} --testlevel ${TESTLEVEL} -r ${Testclass} --verbose --loglevel fatal"
					}
   
					else (rc != 0) 
					{
						error 'Validation failed.'
					}
				}
			}
   		}

        stage('Deploy and Run Tests') 
		{
			if (Deployment_Type=='Deploy Only')
			{	
				script
				{
					if (TESTLEVEL=='NoTestRun') 
					{
						println TESTLEVEL
						rc = command "${toolbelt}/sfdx force:mdapi:deploy -d ${DEPLOYDIR} --wait 10 --targetusername ${SF_USERNAME} "
					}
					else if (TESTLEVEL=='RunLocalTests') 
					{
						println TESTLEVEL
						rc = command "${toolbelt}/sfdx force:mdapi:deploy -d ${DEPLOYDIR} --wait 10 --targetusername ${SF_USERNAME} --testlevel ${TESTLEVEL} --verbose --loglevel fatal"
					}
					else if (TESTLEVEL=='RunSpecifiedTests') 
					{
						println TESTLEVEL
						def Testclass = SpecifyTestClass.replaceAll('\\s','')
						println Testclass						
						rc = command "${toolbelt}/sfdx force:mdapi:deploy -d ${DEPLOYDIR} --wait 10 --targetusername ${SF_USERNAME} --testlevel ${TESTLEVEL} -r ${Testclass} --verbose --loglevel fatal"
					}
					else (rc != 0) 
					{
						error 'Salesforce deployment failed.'
					}
				}
			}
		}
		stage('Delete Components') 
		{
			if (Deployment_Type=='Delete Only')
			{
				rc = command "${toolbelt}/sfdx force:mdapi:deploy -u ${SF_USERNAME} -d ${DELTACHANGES} -w 1"
				
			}
		}

		stage('Delete and Deploy') 
		{
			if (Deployment_Type=='Delete and Deploy')
			{
				script
				{
					if (TESTLEVEL=='NoTestRun') 
					{
						println TESTLEVEL
						rc = command "${toolbelt}/sfdx force:mdapi:deploy -d ${DEPLOYDIR} --wait 10 --targetusername ${SF_USERNAME} "
					}
					else if (TESTLEVEL=='RunLocalTests') 
					{
						println TESTLEVEL
						rc = command "${toolbelt}/sfdx force:mdapi:deploy -d ${DEPLOYDIR} --wait 10 --targetusername ${SF_USERNAME} --testlevel ${TESTLEVEL} --verbose --loglevel fatal"
					}
					else if (TESTLEVEL=='RunSpecifiedTests') 
					{
						println TESTLEVEL
						def Testclass = SpecifyTestClass.replaceAll('\\s','')
						println Testclass						
						rc = command "${toolbelt}/sfdx force:mdapi:deploy -d ${DEPLOYDIR} --wait 10 --targetusername ${SF_USERNAME} --testlevel ${TESTLEVEL} -r ${Testclass} --verbose --loglevel fatal"
					}
					else (rc != 0) 
					{
						error 'Salesforce deployment failed.'
					}
				}
				
			}

		}
		}
    }             
}
def command(script) {
    if (isUnix()) {
        return sh(returnStatus: true, script: script);
    } else {
		return bat(returnStatus: true, script: script);
    }
}
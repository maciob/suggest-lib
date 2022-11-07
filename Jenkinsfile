pipeline 
{
    options 
    {
        timestamps()
    }
    agent any
    tools
    {
        maven 'Maven3.6.2' 
        jdk 'jdk8'
    }
    environment 
    {
        
        MAIN = 'FALSE'
        TAG = ''
        VERSION = ''
        LAST_TAG = ''
        LAST_SUBTAG = ''
        TAGER = ''
    }
    stages 
    {
        stage("Checkout") 
        {
            steps 
            {
                script 
                {
                    STAGE = 'Checkout'
                }
//                updateGitlabCommitStatus name: 'Checkout', state: 'pending'
                deleteDir()
                checkout scm
                script
                {
                    
                    CURRENT_BRANCH = sh(returnStdout: true, script:'git log --all --decorate | head -1 | cut -d "(" -f2 | cut -d ")" -f1 | cut -d "/" -f2-3 | cut -d "," -f1').trim()
                    sh "echo '${CURRENT_BRANCH}'"
                    if ("${CURRENT_BRANCH}".contains("main"))
                    {
                        sh "echo 'TRUE'"
                        MAIN = 'TRUE'
                    }
                    else
                    {
                        sh "echo '${CURRENT_BRANCH}'"
                        try
                        {
                            sh "git checkout '${CURRENT_BRANCH}'"
                        }
                        catch(exc)
                        {
                            sh "git checkout -b '${CURRENT_BRANCH}'"
                        }
                    }
                    CURR_BRANCH_VERSION = sh(returnStdout: true, script:'git log --all --decorate | head -1 | cut -d "(" -f2 | cut -d ")" -f1 | cut -d "/" -f4 | cut -d "," -f1').trim()
                    LAST_TAG = sh(returnStdout: true, script:"git tag | tail -1").trim()
                    VERSION = sh(returnStdout: true, script:"git tag | tail -1 | cut -d . -f1-2").trim()
                    LAST_SUBTAG = sh(returnStdout: true, script:"git tag | tail -1 | cut -d . -f3").trim()
                    SUBTAG = sh(returnStdout: true, script:'echo "$(($(git tag | tail -1 | cut -d . -f3)+1))"').trim()
                    TAG = sh(returnStdout: true, script:"echo '${VERSION}.${SUBTAG}'").trim()
                    sh "echo '${CURR_BRANCH_VERSION}'"
                    sh "echo '${LAST_TAG}'"
                    sh "echo '${VERSION}'"
                    sh "echo '${LAST_SUBTAG}'"
                    sh "echo '${SUBTAG}'"
                    sh "echo '${TAG}'"
                    if ("${CURRENT_BRANCH}".contains("main"))
                    {
                        TAGER = sh(returnStdout: true, script:"echo '${VERSION}.${LAST_SUBTAG}-SNAPSHOT'").trim()
                    }
                    else
                    {
                        SUBTAG = sh(returnStdout: true, script:'echo "$(($(git tag | grep $(git log --all --decorate | head -1 | cut -d "(" -f2 | cut -d ")" -f1 | cut -d / -f4 | cut -d , -f1) | tail -1 | cut -d . -f3)+1))"').trim()
                        TAGER = sh(returnStdout: true, script:"echo '${CURR_BRANCH_VERSION}.${SUBTAG}'").trim()
                    }
                }

//                updateGitlabCommitStatus name: 'Checkout', state: 'success'
            }
        }
        stage('Compile') 
        {
            steps 
            {
//                updateGitlabCommitStatus name: 'Build', state: 'pending'
                script 
                {
                    STAGE = 'Build'
                    sh "mvn compile"
                }
//                updateGitlabCommitStatus name: 'Build', state: 'success'
            }
        }
        stage('Test') 
        {
            steps 
            {
//                updateGitlabCommitStatus name: 'Test', state: 'pending'
                script 
                {
                    STAGE = 'Test'
                    sh "mvn test"
                }
//                updateGitlabCommitStatus name: 'Test', state: 'success'
            }
        }
        stage('Deploy') 
        {
            steps 
            {
//                updateGitlabCommitStatus name: 'Deploy', state: 'pending'

                script 
                {
                    STAGE = 'Deploy'

                    configFileProvider([configFile(fileId: '8412b7f1-a630-4f33-9b49-fe3e1de2ad03',variable: 'MAVEN_SETTINGS_XML')]) 
                    {
                        sh "mvn versions:set -s $MAVEN_SETTINGS_XML -DnewVersion=${TAGER}"
                        sh "mvn deploy -s $MAVEN_SETTINGS_XML"
                    }
                }
//                updateGitlabCommitStatus name: 'Deploy', state: 'success'
            }
        }
        stage('Deploy to GIT')
        {
            when 
            {
                expression { "${MAIN}" == 'FALSE' }
            }
            steps 
            {
                script 
                {
                    sh "echo '${TAGER}'"
                    STAGE = 'Deploy to GIT'
                }
//                updateGitlabCommitStatus name: 'Deploy to GIT', state: 'pending'
                sh 'git config --global user.email "jenkins@jenkins.jenkins"'
                sh 'git config --global user.name "jenkins"'
                sh 'git add .'
                sh "git commit -m '[ci-skip] ${TAGER}'"
                sh "git tag '${TAGER}'"
                withCredentials([string(credentialsId: 'gitlab_api_token', variable: 'TOKEN')]) { 
                    sh 'git push http://Maciob:$TOKEN@3.11.200.189/Maciob/suggest_lib'   
                    sh "git push http://Maciob:$TOKEN@3.11.200.189/Maciob/suggest_lib refs/tags/${TAGER}"
                }
//                updateGitlabCommitStatus name: 'Deploy to GIT', state: 'success'
            }
        }
    }
}

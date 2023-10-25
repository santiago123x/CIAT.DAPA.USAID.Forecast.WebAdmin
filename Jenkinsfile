// Define an empty map for storing remote SSH connection parameters
def remote = [:]

pipeline {

    agent any

    environment {
        user = credentials('aclimate_user')
        host = credentials('aclimate_host')
        name = credentials('aclimate_name')
        ssh_key = credentials('aclimate_devops')
    }

    stages {
        stage('Ssh to connect Melisa server') {
            steps {
                script {
                    // Set up remote SSH connection parameters
                    remote.allowAnyHosts = true
                    remote.identityFile = ssh_key
                    remote.user = user
                    remote.name = name
                    remote.host = host
                    
                }
            }
        }
        stage('Deploy Aclimate WebAdmin') {
            steps {
                script {
                    sshCommand remote: remote, command: """
                        cd /webapps/
                        sudo /bin/systemctl stop aclimate
                        cp -r aclimate_admin/* aclimate_admin_bk/
                        cp -r aclimate_admin/Data/* backup/Data/
                        rm -r aclimate_admin/*
                        rm -fr releaseWebAdmin.zip
                        curl -LOk https://github.com/santiago123x/CIAT.DAPA.USAID.Forecast.WebAdmin/releases/latest/download/releaseWebAdmin.zip
                        unzip -o releaseWebAdmin.zip
                        rm -fr releaseWebAdmin.zip
                        cp -r publish/WebAdmin/* aclimate_admin/
                        rm -r aclimate_admin/appsettings.json
                        rm -r aclimate_admin/appsettings.Development.json
                        cp -r aclimate_admin_bk/appsettings.json aclimate_admin
                        cp -r aclimate_admin_bk/Data/ aclimate_admin
                        cp -r aclimate_admin_bk/api.log aclimate_admin
                        cp -r aclimate_admin_bk/admin.log aclimate_admin
                        rm -fr publish
                        sudo /bin/systemctl start aclimate
                    """
                }
            }
        }
    }
    
    post {
        failure {
            script {
                    sshCommand remote: remote, command: """
                        cd /webapps/
                        sudo /bin/systemctl stop aclimate
                        rm -fr aclimate_admin/*
                        cp -r aclimate_admin_bk/* aclimate_admin/
                        sudo /bin/systemctl start aclimate
                    """
                }
            }
        }

        success {
            script {
                echo 'everything went very well!!'
            }
        }
    }
 
}
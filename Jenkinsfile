    pipeline {
        agent any

        stages {
            stage('git-checkout') {
                steps {
                    git branch: 'main', url: 'https://github.com/sumzzgit/bookstore.git'
                }
            }
            stage('replace the username and password config values'){
                steps{
                    script{
                        withCredentials([usernamePassword(credentialsId: 'rds-creds', usernameVariable: 'USERNAME', passwordVariable: 'PASSWORD')]) {
                            sh '''
                                sed "s/db_user/$USERNAME/" bookstore/connectDB.php.tmp > bookstore/connectDB.php.tmp1
                                sed "s/\\$db_pass/$PASSWORD/" bookstore/connectDB.php.tmp1 > bookstore/connectDB.php.tmp2
                                sed "s/db_user/$USERNAME/g" bookstore/index.php.tmp > bookstore/index.php.tmp.1
                                sed "s/db_pass/$PASSWORD/g" bookstore/index.php.tmp.1 > bookstore/index.php.tmp.2
                            '''
                        }
                    }
                }
            }
            stage("replace the db hostname in config file"){
                steps{
                    withCredentials([string(credentialsId: 'rds-endpoint', variable: 'SECRET')]) {
                        sh '''
                            sed "s/db_host/$SECRET/g" bookstore/index.php.tmp.2 > bookstore/index.php
                            sed "s/db_host/$SECRET/" bookstore/connectDB.php.tmp2 > bookstore/connectDB.php
                        '''
                    }
                }
            }
            stage("replace the nginx path value"){
                steps{
                    sh '''
                        sed "s|nginx_path|/usr/share/nginx/bookstore|" bookstore/nginx.conf.tmp | sudo tee /home/ansible/nginx_rhel.conf
                        sed "s|nginx_path|/var/www/html/bookstore|" bookstore/nginx_conf_ubuntu.tmp | sudo tee /home/ansible/nginx_ubuntu.conf
                    '''
                }
            }
            stage("copy the application files"){
                steps{
                    sh '''
                        sudo mkdir -p /home/ansible/bookstore
                        sudo cp bookstore/*.php /home/ansible/bookstore/
                        sudo cp bookstore/*.sql /home/ansible/bookstore/
                        sudo cp bookstore/nginx.conf.default /home/ansible/bookstore/
                        sudo cp bookstore/style.css /home/ansible/bookstore/
                        sudo cp -r bookstore/image /home/ansible/bookstore/
                        sudo chown -R ansible:ansible /home/ansible/bookstore 
                    '''
                }
            }
            stage("copy the playbook file"){
                steps{
                    sh '''
                        sudo cp bookstore/bookstore_deploy.yml /home/ansible/
                        sudo chown -R ansible:ansible /home/ansible/bookstore_deploy.yml
                    '''
                }
            }
            stage("run the ansible playbook"){
                steps{
                    sh 'sudo -u ansible ansible-playbook /home/ansible/bookstore_deploy.yml'
                }
            }
        }
    }

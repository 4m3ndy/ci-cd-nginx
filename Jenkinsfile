pipeline {
   agent { node { label 'app-node-01'}}

   stages {
      stage('Checkout') {
         steps {
            checkout([$class: 'GitSCM', branches: [[name: '*/master']], userRemoteConfigs: [[credentialsId: 'ci-deployment-key', url: 'git@github.com:4m3ndy/ci-cd-nginx.git']]])
         }
      }
      
      stage('Build') {
           steps {
            sh returnStdout: true, script: 'composer install'
            sh returnStdout: true, script: "cp -a ${JENKINS_HOME}/configs/env-template .env"
           }
      }
      
      stage('Caching Config'){
        steps {
            sh returnStdout: true, script: 'php artisan config:clear'
            sh returnStdout: true, script: 'php artisan config:cache'
        }
      }
      
      stage('DB Migrations & Seeds') {
        steps {
            script {
                if (params.ENVIRONMENT != "production"){
                  sh returnStdout: true, script: 'php artisan migrate --seed'
                } else {
                  sh returnStdout: true, script: 'php artisan migrate --force --seed'
                }
            }
        }
      }
      
      stage('Test'){
          steps {
              sh returnStdout: true, script: 'php artisan test'
          }
      }
      
      stage('Compile Assets'){
        steps {
            sh returnStdout: true, script: 'php artisan view:clear'
            sh returnStdout: true, script: 'php artisan view:cache'
            
            script {
                if (params.ENVIRONMENT != "production"){
                  sh returnStdout: true, script: 'npm install && npm run dev'
                } else {
                  sh returnStdout: true, script: 'npm install --no-dev && npm run production'
                }
            }
            
        }
      }
      
      stage('Rsyncing'){
        steps {
            sh returnStdout: true, script: 'rsync -avr . /var/www/laravel-app/'
        }
      }
      
   }
}

pipeline {
   agent any

   tools {
      //  Install the Maven version configured as "M3" and add it to the path.
      maven "M3"
   }

   stages {
      stage('BUILD') {
         steps {
            git 'https://github.com/kmasani81/WebGoat.git'

            // Run Maven on a Unix agent.
            sh "cd $WORKSPACE/webgoat-server"
            sh "mvn -Dmaven.test.failure.ignore=true clean package"
         }
      }
      stage('Scans: Master') {
         when { branch 'master' }
         steps {
            parallel(
               SonarQube: {
                  sh "mvn sonar:sonar -Dsonar.host.url=http://sonar.masani.xyz:9000 -Dsonar.login=6a369fd08942a7491e92bf374c8a4f1dce0ad25b"
                  echo "Getting the analysis results .. "
               },
               NexusLifeCycle: {
                  sh "echo 'hello world'"
               }
            )
         }
      }

      stage('Scans: Dev') {
         when { not { branch 'master' } }
         steps {
            sh "mvn sonar:sonar -Dsonar.host.url=http://sonar.masani.xyz:9000 -Dsonar.login=6a369fd08942a7491e92bf374c8a4f1dce0ad25b"
            echo "Getting the analysis results .. "
         }
      }

      stage('Nexus Repo') {
         steps {
            sh "echo 'HELLO WORLD'"
         }
      }

      stage('Docker Build') {
         steps {
            sh "echo 'Running Docker build ..' "
            script {
              SHORT_HASH = sh(script: 'git rev-parse --short HEAD', returnStdout: true).trim()
              DOCKER_RELEASE_TAG = "MYAPP-${SHORT_HASH}"
            }
            echo "DOCKER_RELEASE_TAG:  $DOCKER_RELEASE_TAG"
            sh "cd $WORKSPACE/webgoat-server && /usr/bin/docker build -t kmasani/webgoat:${DOCKER_RELEASE_TAG} ."
         }
      }

      stage('Container Scan') {

         steps {
            sh "echo 'Running Container scan .. ' "
            // sh "cd $WORKSPACE && /opt/devops/tools/inline_scan-v0.6.0 scan -r kmasani/myapp:${DOCKER_RELEASE_TAG}"
            // sh "/usr/bin/python /opt/devops/scripts/parse_anchore_analysis.py --outfile $WORKSPACE/anchore-reports/webgoat-local_latest-vuln.json"

            sh "echo 'Pushing Docker .. ' "
            sh "docker push kmasani/myapp:${DOCKER_RELEASE_TAG}"
         }
      }

      stage('Deploy: DEV') {
         steps {
            sh "echo 'Deploying Docker ..' "
            // sh "/usr/bin/python /opt/devops/scripts/deploy_runner.py ${DOCKER_RELEASE_TAG}"
         }
      }      

   }
}

// TODO: https://github.com/phpstan/phpstan + extensions
pipeline {
  agent none
  stages {
    stage("prepare") {
      steps {
        node("php") {
          timeout(time: 5, unit: "MINUTES"){
            dir("config"){
              sh "ssh-keyscan -H gitlab.com >> ~/.ssh/known_hosts"
              sshagent (credentials: ["gitlab"]) {
                git url: "git@gitlab.com:free-sca-src-cfg/free-sca-slim-cfg.git", changelog: true, poll: true, credentialsId: "gitlab"
              }
              stash name: "phpunit.xml", includes: "phpunit.xml"
              stash name: "phpdox.xml", includes: "phpdox.xml"
              stash name: "phpcs.xml", includes: "phpcs.xml"
              stash name: "build.xml", includes: "build.xml"
            }
          }
          dir("source"){
            git url: "https://github.com/slimphp/Slim", branch: "3.x", changelog: true, poll: true
            stash name: "source", excludes: "example/"
          }
          deleteDir()
        }
      }
    }
    stage("syntax") {
      steps {
        node("php&&ant") {
          unstash "source"
          unstash "build.xml"
          sh "ant prepare"
          sh "ant lint-source"
          sh "ant lint-tests"
          dir("build")
          {
            stash name: "lint-source.log", includes: "lint-source.log"
            stash name: "lint-tests.log", includes: "lint-tests.log"
          }
          deleteDir()
        }
      }
    }
    stage("process") {
      steps {
        parallel (
          "phploc-txt": {
            node ("php&&ant") {
              unstash "source"
              unstash "build.xml"
              sh "ant prepare"
              sh "ant phploc-txt"
              dir("build")
              {
                stash name: "phploc.txt", includes: "phploc.txt"
              }
              deleteDir()
            }
          },
          "phploc-xml": {
            node ("php&&ant") {
              unstash "source"
              unstash "build.xml"
              sh "ant prepare"
              sh "ant phploc-xml"
              dir("build")
              {
                stash name: "phploc.log", includes: "phploc.log"
                stash name: "phploc.csv", includes: "phploc.csv"
                stash name: "phploc.xml", includes: "phploc.xml"
              }              
              deleteDir()
            }
          },
          "pdepend-xml": {
            node ("pdepend&&ant") {
              unstash "source"
              unstash "build.xml"
              sh "ant prepare"
              sh "ant pdepend-xml"
              dir("build")
              {
                stash name: "pdepend.log", includes: "pdepend.log"
                stash name: "jdepend.xml", includes: "jdepend.xml"
                stash name: "dependencies.svg", includes: "dependencies.svg"
                stash name: "overview-pyramid.svg", includes: "overview-pyramid.svg"
              }              
              deleteDir()
            }
          },          
          "phpmd-txt": {
            node ("phpmd&&ant") {
              unstash "source"
              unstash "build.xml"
              sh "ant prepare"
              sh "ant phpmd-txt"
              dir("build")
              {
                sh "ls -la"
                stash name: "phpmd.txt", includes: "phpmd.txt"
              }              
              deleteDir()
            }
          },
          "phpmd-xml": {
            node ("phpmd&&ant") {
              unstash "source"
              unstash "build.xml"
              sh "ant prepare"
              sh "ant phpmd-xml"
              dir("build")
              {
                sh "ls -la"
                stash name: "phpmd.log", includes: "phpmd.log"
                stash name: "phpmd.xml", includes: "phpmd.xml"
              }              
              deleteDir()
            }
          },
          "phpcs-txt": {
            node ("phpcs&&ant") {
              unstash "source"
              unstash "build.xml"
              unstash "phpcs.xml"
              sh "ant prepare"
              sh "ant phpcs-txt"
              dir("build")
              {
                stash name: "phpcs.txt", includes: "phpcs.txt"
              }              
              deleteDir()
            }
          },
          "phpcs-xml": {
            node ("phpcs&&ant") {
              unstash "source"
              unstash "build.xml"
              unstash "phpcs.xml"
              sh "ant prepare"
              sh "ant phpcs-xml"
              dir("build")
              {
                stash name: "phpcs.log", includes: "phpcs.log"
                stash name: "checkstyle.xml", includes: "checkstyle.xml"
              }              
              deleteDir()
            }
          },
          "phpcpd-txt": {
            node ("phpcpd&&ant") {
              unstash "source"
              unstash "build.xml"
              sh "ant prepare"
              sh "ant phpcpd-txt"
              dir("build")
              {
                stash name: "phpcpd.txt", includes: "phpcpd.txt"
              }              
              deleteDir()
            }
          },
          "phpcpd-xml": {
            node ("phpcpd&&ant") {
              unstash "source"
              unstash "build.xml"
              sh "ant prepare"
              sh "ant phpcpd-xml"
              dir("build")
              {
                stash name: "phpcpd.log", includes: "phpcpd.log"
                stash name: "phpcpd.xml", includes: "phpcpd.xml"
              }              
              deleteDir()
            }
          },
          "phpunit": {
            node ("phpunit&&ant") {
              unstash "source"
              unstash "build.xml"
              unstash "phpunit.xml"
              sh "ant prepare"
              sh "ant phpunit"
              dir("build")
              {
                stash name: "phpunit.log", includes: "phpunit.log"
                stash name: "clover.xml", includes: "clover.xml"
                stash name: "crap4j.xml", includes: "crap4j.xml"
                stash name: "junit.xml", includes: "junit.xml"
                dir("coverage") {
                 stash name: "coverage"
                }
              }              
              deleteDir()
            }
          },
          "phpdox": {
            node ("phpunit&&ant") {
              echo "fetch from local scm ..."
              unstash "source"
              unstash "build.xml"
              unstash "phpdox.xml"
              sh "ant prepare"
              sh "ant phpdox"
              dir("build")
              {
                stash name: "phpdox.log", includes: "phpdox.log"
                dir("phpdox") {
                 stash name: "phpdox"
                }
                dir("api") {
                 stash name: "api"
                }
              }              
              deleteDir()
            }
          }
        )
      }  
    }
    stage("publish") {
      steps {
        parallel (
          "archive": {
            node ("ant") {
              echo "unstash result  data ..."
              dir("output"){
                unstash "lint-source.log"
                unstash "lint-tests.log"
                unstash "phploc.log"
                unstash "phploc.csv"
                unstash "phploc.xml"
                unstash "phploc.txt"
                unstash "jdepend.xml"
                unstash "pdepend.log"
                unstash "dependencies.svg"
                unstash "overview-pyramid.svg"
                unstash "phpmd.log"
                unstash "phpmd.txt"
                unstash "phpmd.xml"
                unstash "phpcs.log"
                unstash "phpcs.txt"
                unstash "checkstyle.xml"
                unstash "phpcpd.log"
                unstash "phpcpd.txt"
                unstash "phpcpd.xml"
                unstash "clover.xml"
                unstash "crap4j.xml"
                unstash "junit.xml"
                unstash "phpdox.log"
              }
              dir("coverage"){
                unstash "coverage"
              }
              unstash "phpunit.log"
              dir("phpdox")
              {
                unstash "phpdox"
              }
              dir("api")
              {
                unstash "api"
              }
              archiveArtifacts artifacts: "output/phpmd.*", fingerprint: true
              archiveArtifacts artifacts: "output/phpcs.*,output/checkstyle.xml", fingerprint: true
              archiveArtifacts artifacts: "output/phpcpd.*", fingerprint: true
              archiveArtifacts artifacts: "output/phploc.*", fingerprint: true
            }
          },
          "warnings": {
            node ("sca") {
              unstash "lint-source.log"
              step([
                $class: 'WarningsPublisher',
                canComputeNew: true,
                canResolveRelativePaths: false,
                defaultEncoding: '',
                excludePattern: '',
                healthy: '',
                includePattern: '',
                messagesPattern: '',
                parserConfigurations: [[parserName: 'PHP Runtime', pattern: 'lint-source.log']], unHealthy: ''
              ])
            }
          },
          "checkstyle": {
            node ("sca") {
              unstash "checkstyle.xml"
              step([
                $class: 'CheckStylePublisher',
                pattern: '**/checkstyle.xml', 
                unstableTotalAll: '0', 
                usePreviousBuildAsReference: true
              ])
            }
          },          
          "phpmd": {
            node ("sca") {
              unstash "source"
              unstash "phpmd.xml"
              unstash "build.xml"
              sh "ls -la"
              sh "pwd"
              step([
                $class: 'PmdPublisher',
                pattern: '**/phpmd.xml', 
                unstableTotalAll: '0', 
                usePreviousBuildAsReference: true
              ])
              // deleteDir()
            }
          },          
          "phpcpd": {
            node ("sca") {
              unstash "source"
              unstash "phpcpd.xml"
              sh "ls -la"
              step([
                $class: 'DryPublisher',
                pattern: '**/phpcpd.xml', 
                unstableTotalAll: '0', 
                usePreviousBuildAsReference: true
              ])
            }
          },          
          "phpdox": {
            node ("ant") {    
              dir("phpdox")
              {
                unstash "phpdox"
              }
              dir("api")
              {
                unstash "api"
              }
              publishHTML (target: [
                  allowMissing: false,
                  alwaysLinkToLastBuild: false,
                  keepAll: true,
                  reportDir: 'api',
                  reportFiles: 'index.html',
                  reportName: "API Documentation (phpdox)"
              ])          
              deleteDir()
            }
          },
        ) 
      }
    }
  }
}

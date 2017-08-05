pipeline {
  agent none
  stages {
    stage("prepare") {
      steps {
        node("php") {
          sh "ls -la"
          echo "fetch config from scs ..."
          timeout(time: 5, unit: "MINUTES"){
            dir("config"){
              sh "ssh-keyscan -H gitlab.com >> ~/.ssh/known_hosts"
              sshagent (credentials: ["gitlab"]) {
                git url: "git@gitlab.com:free-sca-src-cfg/free-sca-slim-cfg.git", changelog: true, poll: true, credentialsId: "gitlab"
              }
              sh "ls -la"
              stash name: "phpunit.xml", includes: "phpunit.xml"
              stash name: "phpdox.xml", includes: "phpdox.xml"
              stash name: "phpcs.xml", includes: "phpcs.xml"
              stash name: "build.xml", includes: "build.xml"
            }
          }
          echo "saving configs"
          echo "fetch source from scm ..."
          dir("source"){
            git url: "https://github.com/slimphp/Slim", branch: "3.x", changelog: true, poll: true
            sh "ls -la"
            stash excludes: 'example/', name: 'source'
          }
          deleteDir()
          sh "ls -la"
        }
      }
    }
    stage("syntax") {
      steps {
        node("php&&ant") {
          echo "fetch from local scm ..."
          unstash "source"
          unstash "build.xml"
          echo "check syntax ..."
          sh "ant prepare"
          sh "ant lint-source"
          sh "ant lint-tests"
          echo "save report ..."
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
              echo "fetch from local scm ..."
              unstash "source"
              unstash "build.xml"
              echo "phploc txt process ..."
              sh "ant prepare"
              sh "ant phploc-txt"
              sh "ls -la"
              dir("build")
              {
                stash name: "phploc.txt", includes: "phploc.txt"
              }
              deleteDir()
            }
          },
          "phploc-xml": {
            node ("php&&ant") {
              echo "fetch from local scm ..."
              unstash "source"
              unstash "build.xml"
              echo "phploc xml process ..."
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
              echo "fetch from local scm ..."
              unstash "source"
              unstash "build.xml"
              echo "pdepend xml process ..."
              sh "ant prepare"
              sh "ant pdepend-xml"
              echo "save results ..."
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
              echo "fetch from local scm ..."
              unstash "source"
              unstash "build.xml"
              echo "phpmd txt process ..."
              sh "ant prepare"
              sh "ant phpmd-txt"
              echo "save results ..."
              dir("build")
              {
                stash name: "phpmd.txt", includes: "phpmd.txt"
              }              
              deleteDir()
            }
          },
          "phpmd-xml": {
            node ("phpmd&&ant") {
              echo "fetch from local scm ..."
              unstash "source"
              unstash "build.xml"
              echo "phpmd xml process ..."
              sh "ant prepare"
              sh "ant phpmd-xml"
              echo "save results ..."
              dir("build")
              {
                stash name: "phpmd.log", includes: "phpmd.log"
                stash name: "phpmd.xml", includes: "phpmd.xml"
              }              
              deleteDir()
            }
          },
          "phpcs-txt": {
            node ("phpcs&&ant") {
              echo "fetch from local scm ..."
              unstash "source"
              unstash "build.xml"
              unstash "phpcs.xml"
              echo "phpsc txt process ..."
              sh "ant prepare"
              sh "ant phpcs-txt"
              echo "save results ..."
              dir("build")
              {
                stash name: "phpcs.txt", includes: "phpcs.txt"
              }              
              deleteDir()
            }
          },
          "phpcs-xml": {
            node ("phpcs&&ant") {
              echo "fetch from local scm ..."
              unstash "source"
              unstash "build.xml"
              unstash "phpcs.xml"
              echo "phpsc txt process ..."
              sh "ant prepare"
              sh "ant phpcs-xml"
              echo "save results ..."
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
              echo "fetch from local scm ..."
              unstash "source"
              unstash "build.xml"
              echo "phpcpd txt process ..."
              sh "ant prepare"
              sh "ant phpcpd-txt"
              echo "save results ..."
              dir("build")
              {
                stash name: "phpcpd.txt", includes: "phpcpd.txt"
              }              
              deleteDir()
            }
          },
          "phpcpd-xml": {
            node ("phpcpd&&ant") {
              echo "fetch from local scm ..."
              unstash "source"
              unstash "build.xml"
              echo "phpcpd txt process ..."
              sh "ant prepare"
              sh "ant phpcpd-xml"
              echo "save results ..."
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
              echo "fetch from local scm ..."
              unstash "source"
              unstash "build.xml"
              unstash "phpunit.xml"
              echo "phpunit process ..."
              sh "ls -la"
              sh "ant prepare"
              sh "ant phpunit"
              echo "save results ..."
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
              echo "phpdox process ..."
              sh "ant prepare"
              sh "ant phpdox"
              echo "save results ..."
              sh "ls -la"
              sh "ls -la ./build "
              sh "ls -la ./build/api"
              sh "ls -la ./build/phpdox"
              dir("build")
              {
                echo "stash log"
                stash name: "phpdox.log", includes: "phpdox.log"
                echo "stash dox"
                dir("phpdox") {
                 stash name: "phpdox"
                }
                echo "stash api"
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
          "report1": {
            node ("ant") {
              echo "unstash result  data ..."
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
              unstash "phpdox.log"
              sh "ls -la"
              sh "ls -la ./coverage"
              sh "ls -la ./phpdox"
              sh "ls -la ./api"
              echo "build artefacts ..."
            }
          },
          "warnings": {
            node ("php&&ant") {
              /*
              step([
                $class: 'WarningsPublisher',
                canComputeNew: false,
                canResolveRelativePaths: false,
                defaultEncoding: '',
                excludePattern: '',
                healthy: '',
                includePattern: '',
                messagesPattern: '',
                parserConfigurations: [[parserName: 'GNU Make + GNU C Compiler (gcc)', pattern: 'error_and_warnings.txt']], unHealthy: ''
              ])
              */
              step([
                $class: 'WarningsPublisher',
                canComputeNew: true,
                canResolveRelativePaths: false,
                defaultEncoding: '',
                excludePattern: '',
                healthy: '',
                includePattern: '',
                messagesPattern: '',
                parserConfigurations: [[parserName: 'PHP Runtime']], unHealthy: ''
              ])
            }
          }
          "report2": {
            node ("ant") {    
              echo "unstash result  data ..."
              dir("phpdox")
              {
                unstash "phpdox"
              }
              dir("api")
              {
                unstash "api"
              }
              echo "publish report ..."
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

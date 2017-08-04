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
        node("php" && "ant") {
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
            node ("phploc" && "ant") {
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
            node ("phploc" && "ant") {
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
          "phpdepend-xml": {
            node ("phpdepend" && "ant") {
              echo "fetch from local scm ..."
              unstash "source"
              unstash "build.xml"
              echo "phpdepend xml process ..."
              sh "ant prepare"
              sh "ant phpdepend-xml"
              echo "save results ..."
              dir("build")
              {
                stash name: "jdepend.xml", includes: "jdepend.xml"
                stash name: "dependencies.svg", includes: "dependencies.svg"
                stash name: "overview-pyramid.svg", includes: "overview-pyramid.svg"
              }              
              deleteDir()
            }
          },          
          "phpmd-txt": {
            node ("phpmd" && "ant") {
              echo "fetch from local scm ..."
              unstash "source"
              unstash "build.xml"
              sh "ls -la"
              echo "phpmd txt process ..."
            }
          },
          "phpmd-xml": {
            node ("phpmd" && "ant") {
              echo "fetch from local scm ..."
              unstash "source"
              unstash "build.xml"
              sh "ls -la"
              echo "phpmd xml process ..."
            }
          },
          "phpcs-txt": {
            node ("phpcs" && "ant") {
              echo "fetch from local scm ..."
              unstash "source"
              unstash "build.xml"
              unstash "phpcs.xml"
              sh "ls -la"
              echo "phpsc txt process ..."
            }
          },
          "phpcs-xml": {
            node ("phpcs" && "ant") {
              echo "fetch from local scm ..."
              unstash "source"
              unstash "build.xml"
              unstash "phpcs.xml"
              sh "ls -la"
              echo "phpsc xml process ..."
            }
          },
          "phpcpd-txt": {
            node ("phpcpd" && "ant") {
              echo "fetch from local scm ..."
              unstash "source"
              unstash "build.xml"
              sh "ls -la"
              echo "phpcpd txt process ..."
            }
          },
          "phpcpd-xml": {
            node ("phpcpd" && "ant") {
              echo "fetch from local scm ..."
              unstash "source"
              unstash "build.xml"
              sh "ls -la"
              echo "phpcpd xml process ..."
            }
          },
          "phpunit": {
            node ("phpunit" && "ant") {
              echo "fetch from local scm ..."
              unstash "source"
              unstash "build.xml"
              unstash "phpunit.xml"
              sh "ls -la"
              echo "phpunit process ..."
            }
          },
          "phpdox": {
            node ("phpunit" && "ant") {
              echo "fetch from local scm ..."
              unstash "source"
              unstash "build.xml"
              unstash "phpdox.xml"
              sh "ls -la"
              echo "phpdox process ..."
            }
          }
        )
      }  
    }
    stage("publish") {
      steps {
        node("php") {
          echo "unstash result  data ..."
          unstash "lint-source.log"
          unstash "lint-tests.log"
          unstash "phploc.log"
          unstash "phploc.csv"
          unstash "phploc.xml"
          unstash "phploc.txt"
          unstash "jdepend.xml"
          unstash "dependencies.svg"
          unstash "overview-pyramid.svg"
          sh "ls -la"
          echo "build artefacts ..."
          echo "publish artefacts ..."
        }
      }
    }
  }
}

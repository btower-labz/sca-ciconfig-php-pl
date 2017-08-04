pipeline {
  agent none
  stages {
    stage("prepare") {
      steps {
        node("php") {
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
          echo "fetch source from scm ..."
          dir("source"){
            git url: "https://github.com/slimphp/Slim", branch: "3.x", changelog: true, poll: true
            sh "ls -la"
            stash excludes: 'example/', name: 'source'
          }
        }
      }
    }
    stage("syntax") {
      steps {
        node("php" && "ant") {
          echo "fetch from local scm ..."
          unstash "source"
          unstash "build.xml"
          sh "ls -la"
          echo "check syntax ..."
          sh "ant"
          echo "save report ..."
        }
      }
    }
    stage("process") {
      steps {
        parallel (
          "phploc-txt": {
            node ("php") {
              echo "fetch from local scm ..."
              unstash "source"
              sh "ls -la"
              echo "phploc txt process ..."
            }
          },
          "phploc-xml": {
            node ("php") {
              echo "fetch from local scm ..."
              unstash "source"
              sh "ls -la"
              echo "phploc xml process ..."
            }
          },
          "phpdepend-txt": {
            node ("php") {
              echo "fetch from local scm ..."
              unstash "source"
              sh "ls -la"
              echo "phpdepend txt process ..."
            }
          },
          "phpdepend-xml": {
            node ("php") {
              echo "fetch from local scm ..."
              unstash "source"
              sh "ls -la"
              echo "phpdepend xml process ..."
            }
          },          
          "phpmd-txt": {
            node ("php") {
              echo "fetch from local scm ..."
              unstash "source"
              sh "ls -la"
              echo "phpmd txt process ..."
            }
          },
          "phpmd-xml": {
            node ("php") {
              echo "fetch from local scm ..."
              unstash "source"
              sh "ls -la"
              echo "phpmd xml process ..."
            }
          },
          "phpcs-txt": {
            node ("php") {
              echo "fetch from local scm ..."
              unstash "source"
              unstash "phpcs.xml"
              sh "ls -la"
              echo "phpsc txt process ..."
            }
          },
          "phpcs-xml": {
            node ("php") {
              echo "fetch from local scm ..."
              unstash "source"
              unstash "phpcs.xml"
              sh "ls -la"
              echo "phpsc xml process ..."
            }
          },
          "phpcpd-txt": {
            node ("php") {
              echo "fetch from local scm ..."
              unstash "source"
              sh "ls -la"
              echo "phpcpd txt process ..."
            }
          },
          "phpcpd-xml": {
            node ("php") {
              echo "fetch from local scm ..."
              unstash "source"
              sh "ls -la"
              echo "phpcpd xml process ..."
            }
          },
          "phpunit": {
            node ("php") {
              echo "fetch from local scm ..."
              unstash "source"
              unstash "phpunit.xml"
              sh "ls -la"
              echo "phpunit process ..."
            }
          },
          "phpdox": {
            node ("php") {
              echo "fetch from local scm ..."
              unstash "source"
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
          echo "build artefacts ..."
          echo "publish artefacts ..."
        }
      }
    }
  }
}

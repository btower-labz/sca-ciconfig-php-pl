pipeline {
  agent none
  stages {
    stage("prepare") {
      steps {
        node("php") {
          echo "fetch config from scs ..."
          dir("config"){
            git url: "git@gitlab.com:free-sca-src-cfg/free-sca-slim-cfg.git", changelog: true, poll: true
            sh "ls -la"
            stash name: 'config'
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
        node("php") {
          unstash "source"
          sh "ls -la"
          echo "check syntax ..."
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
              echo "phploc txt process ..."
            }
          },
          "phploc-xml": {
            node ("php") {
              echo "fetch from local scm ..."
              echo "phploc xml process ..."
            }
          },
          "phpdepend-txt": {
            node ("php") {
              echo "fetch from local scm ..."
              echo "phpdepend txt process ..."
            }
          },
          "phpdepend-xml": {
            node ("php") {
              echo "fetch from local scm ..."
              echo "phpdepend xml process ..."
            }
          },          
          "phpmd-txt": {
            node ("php") {
              echo "fetch from local scm ..."
              echo "phpmd txt process ..."
            }
          },
          "phpmd-xml": {
            node ("php") {
              echo "fetch from local scm ..."
              echo "phpmd xml process ..."
            }
          },
          "phpcs-txt": {
            node ("php") {
              echo "fetch from local scm ..."
              echo "phpsc txt process ..."
            }
          },
          "phpcs-xml": {
            node ("php") {
              echo "fetch from local scm ..."
              echo "phpsc xml process ..."
            }
          },
          "phpcpd-txt": {
            node ("php") {
              echo "fetch from local scm ..."
              echo "phpcpd txt process ..."
            }
          },
          "phpcpd-xml": {
            node ("php") {
              echo "fetch from local scm ..."
              echo "phpcpd xml process ..."
            }
          },
          "phpunit": {
            node ("php") {
              echo "fetch from local scm ..."
              echo "phpunit process ..."
            }
          },
          "phpdox": {
            node ("php") {
              echo "fetch from local scm ..."
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

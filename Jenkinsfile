// TODO: https://github.com/phpstan/phpstan + extensions
// TODO: https://github.com/sebastianbergmann/phpdcd
// TODO: violations https://issues.jenkins-ci.org/browse/JENKINS-26039
// TODO: https://wiki.jenkins.io/display/JENKINS/JDepend+Plugin
pipeline {
  agent none
  stages {
    stage('prepare') {
      steps {
        node('php') {
          timeout(time: 5, unit: 'MINUTES'){
            dir('config'){
              sh 'ssh-keyscan -H gitlab.com >> ~/.ssh/known_hosts'
              sshagent (credentials: ['gitlab']) {
                git url: 'git@gitlab.com:free-sca-src-cfg/free-sca-slim-cfg.git', changelog: true, poll: true, credentialsId: 'gitlab'
              }
              stash name: 'phpunit.xml', includes: 'phpunit.xml'
              stash name: 'phpdox.xml', includes: 'phpdox.xml'
              stash name: 'phpcs.xml', includes: 'phpcs.xml'
              stash name: 'build.xml', includes: 'build.xml'
            }
          }
          dir('source'){
            git url: 'https://github.com/slimphp/Slim', branch: '3.x', changelog: true, poll: true
            //sh 'cp ./Slim/Route.php ./Slim/Route2.php'
            stash name: 'source', excludes: 'example/'
          }
          deleteDir()
        }
      }
    }
    stage('syntax') {
      steps {
        node('php&&ant') {
          unstash 'source'
          unstash 'build.xml'
          sh 'ant prepare'
          sh 'ant lint-source'
          sh 'ant lint-tests'
          dir('build')
          {
            stash name: 'lint-source.log', includes: 'lint-source.log'
            stash name: 'lint-tests.log', includes: 'lint-tests.log'
          }
          deleteDir()
        }
      }
    }
    stage('process') {
      steps {
        parallel (
          'taskspub': {
            node ('ant') {
              unstash 'source'
              step([
                $class: 'TasksPublisher',
                pattern:  '**/*.php',
                high: 'FIXME, XXX, BUG',
                normal: 'TODO',
                low: 'LATER',
                ignoreCase: true,
                asRegexp: false,
                unstableTotalAll: '0', 
                usePreviousBuildAsReference: true,
              ])
              deleteDir()
            }
          },
          'phploc-txt': {
            node ('php&&ant') {
              unstash 'source'
              unstash 'build.xml'
              sh 'ant prepare'
              sh 'ant phploc-txt'
              dir('build')
              {
                stash name: 'phploc.txt', includes: 'phploc.txt'
              }
              deleteDir()
            }
          },
          'phploc-xml': {
            node ('php&&ant') {
              unstash 'source'
              unstash 'build.xml'
              sh 'ant prepare'
              sh 'ant phploc-xml'
              dir('build')
              {
                stash name: 'phploc.log', includes: 'phploc.log'
                stash name: 'phploc.csv', includes: 'phploc.csv'
                stash name: 'phploc.xml', includes: 'phploc.xml'
              }              
              deleteDir()
            }
          },
          'pdepend-xml': {
            node ('pdepend&&ant') {
              unstash 'source'
              unstash 'build.xml'
              sh 'ant prepare'
              sh 'ant pdepend-xml'
              dir('build')
              {
                stash name: 'pdepend.log', includes: 'pdepend.log'
                stash name: 'jdepend.xml', includes: 'jdepend.xml'
                stash name: 'dependencies.svg', includes: 'dependencies.svg'
                stash name: 'overview-pyramid.svg', includes: 'overview-pyramid.svg'
              }              
              deleteDir()
            }
          },          
          'phpmd-txt': {
            node ('phpmd&&ant') {
              unstash 'source'
              unstash 'build.xml'
              sh 'ant prepare'
              sh 'ant phpmd-txt'
              dir('build')
              {
                sh 'ls -la'
                stash name: 'phpmd.txt', includes: 'phpmd.txt'
              }              
              deleteDir()
            }
          },
          'phpmd-xml': {
            node ('phpmd&&ant') {
              unstash 'source'
              unstash 'build.xml'
              sh 'ant prepare'
              sh 'ant phpmd-xml'
              dir('build')
              {
                stash name: 'phpmd.log', includes: 'phpmd.log'
                stash name: 'phpmd.xml', includes: 'phpmd.xml'
              }
              // TODO: set threshholds
              // TODO: move parallel, use xmlstarlet
              pmd pattern: '**/build/phpmd.xml', unstableTotalAll: '0', usePreviousBuildAsReference: true
              /*
              step([
                $class: 'PmdPublisher',
                pattern: ' * * /build/phpmd.xml', 
                unstableTotalAll: '0', 
                usePreviousBuildAsReference: true
              ])
              */
              deleteDir()
            }
          },
          'phpmd-html': {
            node ('phpmd&&ant') {
              unstash 'source'
              unstash 'build.xml'
              sh 'ant prepare'
              sh 'ant phpmd-html'
              dir('build')
              {
                stash name: 'phpmd-html.log', includes: 'phpmd-html.log'
                stash name: 'phpmd.html', includes: 'phpmd.html'
              }              
              deleteDir()
            }
          },
          'phpcs-txt': {
            node ('phpcs&&ant') {
              unstash 'source'
              unstash 'build.xml'
              unstash 'phpcs.xml'
              sh 'ant prepare'
              sh 'ant phpcs-txt'
              dir('build')
              {
                stash name: 'phpcs.txt', includes: 'phpcs.txt'
              }              
              deleteDir()
            }
          },
          'phpcs-xml': {
            node ('phpcs&&ant') {
              unstash 'source'
              unstash 'build.xml'
              unstash 'phpcs.xml'
              sh 'ant prepare'
              sh 'ant phpcs-xml'
              dir('build')
              {
                stash name: 'phpcs.log', includes: 'phpcs.log'
                stash name: 'checkstyle.xml', includes: 'checkstyle.xml'
              }              
              deleteDir()
            }
          },
          'phpcpd-txt': {
            node ('phpcpd&&ant') {
              unstash 'source'
              unstash 'build.xml'
              sh 'ant prepare'
              sh 'ant phpcpd-txt'
              dir('build')
              {
                stash name: 'phpcpd.txt', includes: 'phpcpd.txt'
              }              
              deleteDir()
            }
          },
          'phpcpd-xml': {
            node ('phpcpd&&ant') {
              unstash 'source'
              unstash 'build.xml'
              sh 'ant prepare'
              sh 'ant phpcpd-xml'
              dir('build')
              {
                stash name: 'phpcpd.log', includes: 'phpcpd.log'
                stash name: 'phpcpd.xml', includes: 'phpcpd.xml'
              }
              // TODO: move it to parralel (xmlstarlet)
              dry canRunOnFailed: true, pattern: '**/build/phpcpd.xml', unstableTotalAll: '0', usePreviousBuildAsReference: true, highThreshold : 50, normalThreshold : 25
              /*
              step([
                $class: 'DryPublisher',
                pattern:  * * / build/phpcpd.xml', 
                unstableTotalAll: '0', 
                usePreviousBuildAsReference: true,
                highThreshold : 50,
                normalThreshold : 25
              ])
              */
              deleteDir()
            }
          },
          'phpunit': {
            node ('phpunit&&ant') {
              unstash 'source'
              unstash 'build.xml'
              unstash 'phpunit.xml'
              sh 'ant prepare'
              sh 'ant phpunit'
              dir('build')
              {
                stash name: 'phpunit.log', includes: 'phpunit.log'
                stash name: 'clover.xml', includes: 'clover.xml'
                stash name: 'crap4j.xml', includes: 'crap4j.xml'
                stash name: 'junit.xml', includes: 'junit.xml'
                dir('coverage') {
                 stash name: 'coverage'
                }
              }
              
              // TODO: move parallel
              // FIXME: wount work as expected. cannot resolve php packages. Use CloverPHPPublisher when it will  be ready
              step([
                $class: 'CloverPublisher',
                cloverReportDir: './build',
                cloverReportFileName: 'clover.xml',
                healthyTarget: [methodCoverage: 70, conditionalCoverage: 80, statementCoverage: 80], // optional, default is: method=70, conditional=80, statement=80
                unhealthyTarget: [methodCoverage: 50, conditionalCoverage: 50, statementCoverage: 50], // optional, default is none
                failingTarget: [methodCoverage: 0, conditionalCoverage: 0, statementCoverage: 0]     // optional, default is none
              ])
              
              deleteDir()
            }
          },
          'phpdox': {
            node ('phpunit&&ant') {
              echo 'fetch from local scm ...'
              unstash 'source'
              unstash 'build.xml'
              unstash 'phpdox.xml'
              sh 'ant prepare'
              sh 'ant phpdox'
              dir('build')
              {
                stash name: 'phpdox.log', includes: 'phpdox.log'
                dir('phpdox') {
                 stash name: 'phpdox'
                }
                dir('api') {
                 stash name: 'api'
                }
              }              
              deleteDir()
            }
          }
        )
      }  
    }
    stage('publish') {
      steps {
        parallel (
          'archive': {
            node ('ant') {
              echo 'unstash result  data ...'
              dir('output'){
                // php-lint
                unstash 'lint-source.log'
                unstash 'lint-tests.log'
                // php-loc
                unstash 'phploc.log'
                unstash 'phploc.csv'
                unstash 'phploc.xml'
                unstash 'phploc.txt'
                // php-depend
                unstash 'jdepend.xml'
                unstash 'pdepend.log'
                unstash 'dependencies.svg'
                unstash 'overview-pyramid.svg'
                // php-md
                unstash 'phpmd.log'
                unstash 'phpmd.txt'
                unstash 'phpmd.xml'
                unstash 'phpmd.html'
                unstash 'phpmd-html.log'
                // php-cs
                unstash 'phpcs.log'
                unstash 'phpcs.txt'
                unstash 'checkstyle.xml'
                // php-cpd
                unstash 'phpcpd.log'
                unstash 'phpcpd.txt'
                unstash 'phpcpd.xml'
                // php-unit
                unstash 'clover.xml'
                unstash 'crap4j.xml'
                unstash 'junit.xml'
                unstash 'phpunit.log'
                // php-dox
                unstash 'phpdox.log'
                // php-depend
                unstash 'dependencies.svg'
                unstash 'overview-pyramid.svg'
                unstash 'phpdepend.log'                
              }
              dir('coverage'){
                unstash 'coverage'
              }
              unstash 'phpunit.log'
              dir('phpdox')
              {
                unstash 'phpdox'
              }
              dir('api')
              {
                unstash 'api'
              }
              archiveArtifacts artifacts: 'output/lint-source.*', fingerprint: true
              archiveArtifacts artifacts: 'output/phpmd.*', fingerprint: true
              archiveArtifacts artifacts: 'output/phpcs.*,output/checkstyle.xml', fingerprint: true
              archiveArtifacts artifacts: 'output/phpcpd.*', fingerprint: true
              archiveArtifacts artifacts: 'output/phploc.*', fingerprint: true
              archiveArtifacts artifacts: 'output/clover.xml,output/phpunit.log', fingerprint: true
              archiveArtifacts artifacts: 'output/dependencies.svg,output/overview-pyramid.svg', fingerprint: true
            }
          },
          'warnings': {
            node ('sca') {
              unstash 'lint-source.log'
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
          
          // TODO: It works, but we are to use PHPUnitTool
          'junit': {
            node ('sca') {
              unstash 'junit.xml'
              step([
                  $class: 'XUnitBuilder',
                  thresholds: [[$class: 'FailedThreshold', unstableThreshold: '1']],
                  tools: [[$class: 'JUnitType', pattern: '**/junit.xml']]
              ])          
            }
          },          
          
          'checkstyle': {
            node ('sca') {
              unstash 'checkstyle.xml'
              step([
                $class: 'CheckStylePublisher',
                pattern: '**/checkstyle.xml', 
                unstableTotalAll: '0', 
                usePreviousBuildAsReference: true
              ])
            }
          },          
          // TODO: to use it here one need to xmlstarlet phpmd.xml to remove absolute path
          /*
          'phpmd': {
            node ('sca') {
              unstash 'source'
              unstash 'phpmd.xml'
              unstash 'build.xml'
              step([
                $class: 'PmdPublisher',
                pattern: '* * / phpmd.xml', 
                unstableTotalAll: '0', 
                usePreviousBuildAsReference: true
              ])
              deleteDir()
            }
          },
          */
          
          // TODO: xmlstarlet ...
          /*
          'coverage': {
            node ('sca') {
              
              unstash 'clover.xml'
              step([
                $class: 'CloverPublisher',
                cloverReportDir: './',
                cloverReportFileName: './clover.xml',
                healthyTarget: [methodCoverage: 70, conditionalCoverage: 80, statementCoverage: 80], // optional, default is: method=70, conditional=80, statement=80
                unhealthyTarget: [methodCoverage: 50, conditionalCoverage: 50, statementCoverage: 50], // optional, default is none
                failingTarget: [methodCoverage: 0, conditionalCoverage: 0, statementCoverage: 0]     // optional, default is none
              ])          
            }
          },
          */
          // TODO: cannot run here. use xmlstarlet to fix xml
          /*
          'phpcpd': {
            node ('sca') {
              unstash 'source'
              unstash 'phpcpd.xml'
              step([
                $class: 'DryPublisher',
                pattern: ' * * / phpcpd.xml', 
                unstableTotalAll: '0', 
                usePreviousBuildAsReference: true,
                highThreshold : 50,
                normalThreshold : 25
              ])
            }
          },
          */          
          'phpdox': {
            node ('ant') {    
              dir('api')
              {
                unstash 'api'
              }
              publishHTML (target: [
                  allowMissing: false,
                  alwaysLinkToLastBuild: false,
                  keepAll: true,
                  reportDir: 'api',
                  reportFiles: 'index.html',
                  reportName: 'API Documentation (phpdox)'
              ])          
              deleteDir()
            }
          },
          // TODO: too ugly
          'pdependhtml': {
            node ('ant') {    
              dir('pdepend')
              {
                unstash 'dependencies.svg'
                unstash 'overview-pyramid.svg'
                echo "<html><body><p><img src=\"dependencies.svg\" /></p><p><img src=\"overview-pyramid.svg\" /></p.</body></html>"
              }
              publishHTML (target: [
                  allowMissing: false,
                  alwaysLinkToLastBuild: false,
                  keepAll: true,
                  reportDir: 'pdepend',
                  reportFiles: 'index.html',
                  reportName: 'Unit Test Coverage (cloverphp)'
              ])          
              deleteDir()
            }
          },
          'coveragehtml': {
            node ('ant') {    
              dir('coverage')
              {
                unstash 'coverage'
              }
              publishHTML (target: [
                  allowMissing: false,
                  alwaysLinkToLastBuild: false,
                  keepAll: true,
                  reportDir: 'coverage',
                  reportFiles: 'index.html',
                  reportName: 'Unit Test Coverage (cloverphp)'
              ])          
              deleteDir()
            }
          },
        ) 
      }
    }
  }
}

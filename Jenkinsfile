#!/usr/bin/env groovy
@Library(['piper-lib', 'piper-lib-os']) _


pipeline {
    agent { label 'slave' }
    parameters{
        booleanParam(defaultValue: false, description: '\'true\' will create a release artifact on Nexus', name: 'PROMOTE')
    }

    stages{
    /*
        stage('Pull-request voting') {
            when { branch "PR-*" }
            steps {
                script {
                    deleteDir()
                    checkout scm
                    setupPipelineEnvironment script: this
                    measureDuration(script: this, measurementName: 'voter_duration') {
                        sh """
                             echo "add simple tests that should run as part of PR check, e.g. unit tests"
                           """
                    }
                }
            }
            post { always { deleteDir() } }
        }
*/
        /*
        stage('Unit Tests, Coverage & Pylint') {
            agent {
                node {
                    label 'slave'
                    customWorkspace "workspace/${env.JOB_NAME}/${env.BUILD_NUMBER}"
                }
            }
            when {
                anyOf {
                    branch 'master'
                    branch 'PR-*'
                }
            }
            steps {
                script{
                    def customImage = docker.build("mlf_training")
                    customImage.inside(){
                        sh """
                             echo "Making Unit Test Script to Executable"
                             chmod +x jenkins_scripts/unit_test.sh

                             echo "Executing Unit Tests"
                             ./jenkins_scripts/unit_test.sh
                       """
                    }
                    stash includes: 'nosetests.xml', name: 'unit_results'
                    stash includes: 'coverage.xml', name: 'coverage_results'
                    stash includes: 'pylint.out', name: 'pylint_results'

                }
                publishTestResults(
                    junit: [updateResults: true, archive: true, pattern:'nosetests.xml'],
                    cobertura: [archive: true, pattern: 'coverage.xml', onlyStableBuilds: false],
                    allowUnstableBuilds: true
                )
                checksPublishResults(
                    script: this,
                    archive: true,
                    tasks: true,
                    pylint: [pattern: 'pylint.out', thresholds: [fail: [all: '3999', low: '999', normal: '999', high: '50']]],
                    aggregation: [thresholds: [fail: [all: '3999', low: '999', normal: '999', high: '50']]]
                )
            }
            post { always { deleteDir() }  }
        }
        */
        //**********************************************************************************
        // Here is an example, you can implement a different way
        //                           sh """
        //                             chmod +x scripts/run_coverage.sh
        //                             ./scripts/run_coverage.sh tests .
        //                             """
        //                      }
        //                      publishTestResults(
        //                         cobertura: [archive: true, pattern: '**/coverage.xml'],
        //                         allowUnstableBuilds: true
        //                       )
        //**********************************************************************************


        stage('Central Build') {
            when {
                branch 'XAI_NEW'
            }
            steps {
                script{
                    lock(resource: "${env.JOB_NAME}/10", inversePrecedence: true) {
                        milestone 10
                        deleteDir()
                        checkout scm

                        setupPipelineEnvironment script: this
                        measureDuration(script: this, measurementName: 'build_duration') {
                            setVersion script:this
                            stashFiles(script: this) {
                                executeBuild script: this, buildType: 'xMakeStage'
                            }
                            // For additional behavior, please visit: https://github.wdf.sap.corp/pages/ContinuousDelivery/piper-doc/steps/testsPublishResults/
                            //    publishTestResults cobertura: [archive: true, pattern: '**/target/code-coverage/total-coverage.xml']
                        }
                    }
                }
            }
            post { always { deleteDir() } }
        }

        // for additional behavior, please visit: https://github.wdf.sap.corp/pages/ContinuousDelivery/piper-doc/steps/executeSonarScan/
        stage('SonarQube') {
            agent {
                node {
                    label 'slave'
                    customWorkspace "workspace/${env.JOB_NAME}/${env.BUILD_NUMBER}"
                }
            }
            when {
                anyOf {
                    branch 'develop'
                    branch 'XAI_NEW'
                    branch 'PR-*'
                }
            }
            steps {
                script{
                    lock(resource: "${env.JOB_NAME}/20") {
                        milestone 20
                        setupPipelineEnvironment script: this
                        unstash 'unit_results'
                        unstash 'coverage_results'
                        unstash 'pylint_results'
                        def version = readFile('version.txt')
                        measureDuration(script: this, measurementName: 'Sonar_duration') {
                            if(env.BRANCH_NAME.startsWith("PR-")) {

                                echo "SonarQube PR analysis"
                                executeSonarScan script: this, projectVersion: "${version}",
                                    options: "-Dsonar.pullrequest.key=${CHANGE_ID} -Dsonar.pullrequest.branch=${BRANCH_NAME} -Dsonar.pullrequest.base=${CHANGE_TARGET}"

                            } else if(env.BRANCH_NAME == 'develop' || env.BRANCH_NAME == 'release' || env.BRANCH_NAME == 'XAI_NEW') {

                                echo "SonarQube Long Lived Branch Analysis"
                                executeSonarScan script: this, projectVersion: "${version}",
                                    options: "-Dsonar.branch.name=${BRANCH_NAME}"

                            } else {
                                /* It's impossible to happen under current Jenkins configuration */
                                echo "SonarQube Short Lived Branch Analysis -- deprecated"
                            }
                        }
                    }
                }
            }
            post { always { deleteDir() } }
        }


        stage('Deploy to Integration') {
            when { branch 'XAI_NEW' }
            steps {
                lock(resource: "${env.JOB_NAME}/30", inversePrecedence: true) {
                    milestone 30
                    measureDuration(script: this, measurementName: 'deploy_test_duration') {
                        downloadArtifactsFromNexus script: this, fromStaging: true
                        //  deployToCloudFoundry
                    }
                }
            }
            post { always { deleteDir() } }
        }


        stage('Integration tests') {
            when { branch 'XAI_NEW' }
            steps {
                sh """
                                            echo "add your integration tests"
                                           """
            }
            post { always { deleteDir() } }
        }

        stage('Deploy to Performance') {
            when { branch 'XAI_NEW' }
            steps {
                echo "Add deployment to Performance space"
            }
            post { always { deleteDir() } }
        }


        stage('Performance tests') {
            when { branch 'XAI_NEW' }
            steps {
                echo "Add performance tests"
            }
            post { always { deleteDir() } }
        }

        stage('Deploy to Acceptance') {
            when { branch 'XAI_NEW' }
            steps {
                echo "Add deployment to Acceptance space"
            }
            post { always { deleteDir() } }
        }

        stage('Acceptance tests') {
            when { branch 'XAI_NEW' }
            steps {
                echo "Add acceptance tests"
            }
            post { always { deleteDir() } }
        }

        // for additional behavior, please visit: https://github.wdf.sap.corp/pages/ContinuousDelivery/piper-doc/steps/executeWhitesourceScan/
        //       stage('Whitesource') {
        //                 when { branch 'master' }
        //                       steps {
        //                          lock(resource: "${env.JOB_NAME}/40") {
        //                             milestone 40
        //                             measureDuration(script: this, measurementName: 'whitesource_duration') {
        //                              executeWhitesourceScan script: this, scanType: 'unifiedAgent'
        //                     }
        //                  }
        //               }
        //              post { always { deleteDir() } }
        //       }

        // for additional behavior, please visit: https://github.wdf.sap.corp/pages/ContinuousDelivery/piper-doc/steps/executePPMSWhitesourceComplianceCheck/
        //         stage('PPMS Whitesource Compliance') {
        //                  when { branch 'master' }
        //                         steps {
        //                            lock(resource: "${env.JOB_NAME}/50") {
        //                                milestone 50
        //                                measureDuration(script: this, measurementName: 'PPMS_duration') {
        //                                   executePPMSComplianceCheck script: this, scanType: 'whitesource'
        //                      }
        //                   }
        //                }
        //               post { always { deleteDir() } }
        //         }

        // for additional behavior, please visit: https://github.wdf.sap.corp/pages/ContinuousDelivery/piper-doc/steps/executeCheckmarxScan/
        stage('Checkmarx') {
            when { branch 'XAI_NEW' }
            steps {
                lock(resource: "${env.JOB_NAME}/60") {
                    milestone 60
                    measureDuration(script: this, measurementName: 'checkmarx_duration') {
                        executeCheckmarxScan script: this
                    }
                }
            }
            post { always { deleteDir() } }
        }
        // for additional behavior, please visit: https://github.wdf.sap.corp/pages/ContinuousDelivery/piper-doc/steps/executeFortifyScan/
        //           stage('Fortify') {
        //                   when { branch 'master' }
        //                       steps {
        //                        lock(resource: "${env.JOB_NAME}/70") {
        //                           milestone 70
        //                             measureDuration(script: this, measurementName: 'fortify_duration') {
        //                                 executeFortifyScan script: this
        //                             }
        //                         }
        //                     post { always { deleteDir() } }
        //                 }
        //          }
        // for additional behavior please visit: https://github.wdf.sap.corp/pages/ContinuousDelivery/piper-doc/steps/executeVulasScan/
        stage('Vulas') {
            when { branch 'XAI_NEW' }
            steps {
                lock(resource: "${env.JOB_NAME}/80") {
                    milestone 80
                    measureDuration(script: this, measurementName: 'vulas_duration') {
                        executeVulasScan script: this, scanType: 'pip'
                    }
                }
            }
            post { always { deleteDir() } }
        }
        // for more information, please visit: https://github.wdf.sap.corp/pages/ContinuousDelivery/piper-doc/steps/sapCreateTraceabilityReport/
        //           stage('Create traceability report') {
        //                     when { branch 'master' }
        //                            steps {
        //                                  sapCreateTraceabilityReport(
        //                                        deliveryMappingFile: '.pipeline/delivery.mapping',
        //                                        requirementMappingFile: '.pipeline/requirement.mapping',
        //                                        failOnError: false )
        //                            }
        //              }
        // for additional behavior please visit: https://github.wdf.sap.corp/pages/ContinuousDelivery/piper-doc/steps/siriusUploadDocument/
        //          stage('Sirius Document Upload') {
        //                   when { branch 'master' }
        //                       steps {
        //                          script{
        //                               siriusUploadDocument script: this,
        //                                                    siriusCredentialsId: 'SiriusCredentials',
        //                                                    siriusProgramName: 'Test_A',
        //                                                    siriusDeliveryName: 'test1',
        //                                                    siriusTaskGuid: '6CAE8B26E4CB1ED6A0C8D43FBCF8B445',
        //                                                    fileName: 'vulas-report.html'
        //                          }
        //                   }
        //              post { always { deleteDir() }  }
        //           }

        // for additional behavior, please visit: https://github.wdf.sap.corp/pages/ContinuousDelivery/piper-doc/stages/promote/
        stage('Promote') {
            when { branch 'XAI_NEW' }
            steps {
                script{
                    lock(resource: "${env.JOB_NAME}/90", inversePrecedence: true) {
                        milestone 90
                        if ("${params.PROMOTE}" == "true") {
                            measureDuration(script: this, measurementName: 'promote_duration') {
                                executeBuild script: this, buildType: 'xMakePromote'
                            }
                        }
                    }
                }
            }
            post { always { deleteDir() } }
        }
    }
    // Send notification mail when pipeline fails
}
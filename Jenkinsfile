@Library('jenkins-libs') _
import it.loopback.jenkins.Projedit
def projedit = new it.loopback.jenkins.Projedit(this)

pipeline {
    agent any
    options {
        // disabilita la compilazione parallela
        disableConcurrentBuilds()
        // viene tenuto lo storico solo delle ultime 10 compilazioni
        buildDiscarder(logRotator(numToKeepStr: '10'))
        // stampa data/ora nei log
        timestamps()
    }
    // esegue il controllo dei commit
    triggers { 
        pollSCM('H/2 * * * *') 
        githubPush()
    }
    stages {
        stage ('Checkout') { steps { checkout scm  } }

        stage('short circuit') {
            // questo stage viene eseguito solo se l'ultimo commit pushato è stato effetuato da jenkins
            when { expression { test_committer('jenkins')  } }
            steps {
                script {
                    print "skip build"
                    // questa compilazione avrà lo stesso stato della precedente
                    currentBuild.result = currentBuild?.previousBuild?.result
                    currentBuild.keepLog = false
                    // descrizione della corrente build
                    currentBuild.description = "skipped"
                    return
                }
            }
        }

        stage('nuget download windows') {
            agent { node { label 'windows' } }
            when { expression { !test_committer('jenkins')  } }
            steps {
                script {
                    // download dell'eseguibile nuget.exe
                    nuget_download.download_windows()

                    archiveArtifacts artifacts: "nuget.exe", fingerprint: true, onlyIfSuccessful: true
                }
            }
        }

        stage('nuget download linux') {
            agent { node { label 'linux' } }
            when { expression { !test_committer('jenkins')  } }
            steps {
                script {
                    // download dell'eseguibile nuget.exe
                    nuget_download()

                    archiveArtifacts artifacts: "nuget.exe", fingerprint: true, onlyIfSuccessful: true
                }
            }
        }


        // tutti gli altri stage non vengono eseguiti se l'ultimo commit è di jenkins

        stage('increase build version C# windows') {
            agent { node { label 'windows' } }
            when { expression { !test_committer('jenkins')  } }
            steps {
                script {
                    // incremento della versione del file dell'assembly generato
                    env.NEW_VERSION = projedit.projedit("csframework", "AssemblyInfo.cs")
                    // aggiunge la versione alla descrizione della compilazione
                    currentBuild.description = currentBuild.description + " version ${env.NEW_VERSION}"

                    archiveArtifacts artifacts: "AssemblyInfo.cs", fingerprint: true, onlyIfSuccessful: true
                }
            }
        }

        stage('increase build version C# linux') {
            agent { node { label 'linux' } }
            when { expression { !test_committer('jenkins')  } }
            steps {
                script {
                    // incremento della versione del file dell'assembly generato
                    env.NEW_VERSION = projedit.projedit("csframework", "AssemblyInfo.cs")
                    // aggiunge la versione alla descrizione della compilazione
                    currentBuild.description = currentBuild.description + " version ${env.NEW_VERSION}"

                    archiveArtifacts artifacts: "AssemblyInfo.cs", fingerprint: true, onlyIfSuccessful: true
                }
            }
        }

        stage('increase build version VB.NET windows') {
            agent { node { label 'windows' } }
            when { expression { !test_committer('jenkins')  } }
            steps {
                script {
                    // incremento della versione del file dell'assembly generato
                    env.NEW_VERSION = projedit.projedit("vbframework", "AssemblyInfo.vb")
                    // aggiunge la versione alla descrizione della compilazione
                    currentBuild.description = currentBuild.description + " version ${env.NEW_VERSION}"

                    archiveArtifacts artifacts: "AssemblyInfo.vb", fingerprint: true, onlyIfSuccessful: true
                }
            }
        }

        stage('increase build version VB.NET linux') {
            agent { node { label 'linux' } }
            when { expression { !test_committer('jenkins')  } }
            steps {
                script {
                    // incremento della versione del file dell'assembly generato
                    env.NEW_VERSION = projedit.projedit("vbframework", "AssemblyInfo.vb")
                    // aggiunge la versione alla descrizione della compilazione
                    currentBuild.description = currentBuild.description + " version ${env.NEW_VERSION}"

                    archiveArtifacts artifacts: "AssemblyInfo.cs", fingerprint: true, onlyIfSuccessful: true
                }
            }
        }
        
        stage('write_build_info windows') {
            agent { node { label 'windows' } }
            when { expression { !test_committer('jenkins')  } }
            steps {
                script {
                    write_build_info.to_json_file("write_build_info_w.json")
                    write_build_info.to_yaml_file("write_build_info_w.yaml")

                    archiveArtifacts artifacts: "write_build_info_w.*", fingerprint: true, onlyIfSuccessful: true
                }
            }
        }

        stage('write_build_info linux') {
            agent { node { label 'linux' } }
            when { expression { !test_committer('jenkins')  } }
            steps {
                script {
                    write_build_info.to_json_file("write_build_info_l.json")
                    write_build_info.to_yaml_file("write_build_info_l.yaml")
                    
                    archiveArtifacts artifacts: "write_build_info_l.*", fingerprint: true, onlyIfSuccessful: true
                }
            }
        }

        
        stage('push changes') {
            when { expression { !test_committer('jenkins')  } }
            steps {
                script {
                    // commit e push delle modifiche ( salvataggio della versione incrementata )
                    env.J_CREDS_IDS = '	repo-git' // credenziali che hanno accesso al repositori
                    env.J_GIT_CONFIG = 'false'
                    env.J_USERNAME = 'jenkins'
                    env.J_EMAIL = 'jenkins@loopback.it'
                    env.J_GIT_CONFIG = "true"
                    env.BRANCH_NAME = "master" // branch remoto sul quale pushare le modifiche
                    git_push_ssh commitMsg: "Jenkins build #${env.BUILD_NUMBER}", tagName: "${env.NEW_VERSION}", files: "."
                }
            }
        }
    }
    post { 
        always { 
            cleanWs()
        }
    }
}

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
        //pollSCM(scmpoll_spec: 'H/2 * * * *', ignorePostCommitHooks: true)
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
        // tutti gli altri stage non vengono eseguiti se l'ultimo commit è di jenkins

        stage('write_build_info linux') {
            agent { node { label 'linux' } }
            when { expression { !test_committer('jenkins')  } }
            steps {
                script {
                    write_build_info.to_json_file("write_build_info_l.json")
                    write_build_info.to_yaml_file("write_build_info_l.yaml")
                    
                    archiveArtifacts artifacts: "write_build_info_l.*", fingerprint: true, onlyIfSuccessful: true

                    stash includes: 'write_build_info_l.*', name: 'write_build_info_l'
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

                    stash includes: 'write_build_info_w.*', name: 'write_build_info_w'
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
                    powershell 'nuget help'
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
                    sh 'mono nuget.exe help'
                    archiveArtifacts artifacts: "nuget.exe", fingerprint: true, onlyIfSuccessful: true
                }
            }
        }

        stage('increase build version C# windows') {
            agent { node { label 'windows' } }
            when { expression { !test_committer('jenkins')  } }
            steps {
                script {
                    // incremento della versione del file dell'assembly generato
                    env.NEW_VERSION = projedit.increase_version("csframework", "AssemblyInfo.cs")
                    // aggiunge la versione alla descrizione della compilazione
                    currentBuild.description = currentBuild.description + " version ${env.NEW_VERSION}"

                    archiveArtifacts artifacts: "AssemblyInfo.cs", fingerprint: true, onlyIfSuccessful: true

                    stash includes: 'AssemblyInfo.cs', name: 'AssemblyInfo.cs'
                }
            }
        }

        stage('increase build version C# linux') {
            agent { node { label 'linux' } }
            when { expression { !test_committer('jenkins')  } }
            steps {
                script {
                    // incremento della versione del file dell'assembly generato
                    env.NEW_VERSION = projedit.increase_version("csframework", "AssemblyInfo.cs")
                    // aggiunge la versione alla descrizione della compilazione
                    currentBuild.description = currentBuild.description + " version ${env.NEW_VERSION}"

                    archiveArtifacts artifacts: "AssemblyInfo.cs", fingerprint: true, onlyIfSuccessful: true

                    stash includes: 'AssemblyInfo.cs', name: 'AssemblyInfo.cs'
                }
            }
        }

        stage('increase build version VB.NET windows') {
            agent { node { label 'windows' } }
            when { expression { !test_committer('jenkins')  } }
            steps {
                script {
                    // incremento della versione del file dell'assembly generato
                    env.NEW_VERSION = projedit.increase_version("vbframework", "AssemblyInfo.vb")
                    // aggiunge la versione alla descrizione della compilazione
                    currentBuild.description = currentBuild.description + " version ${env.NEW_VERSION}"

                    archiveArtifacts artifacts: "AssemblyInfo.vb", fingerprint: true, onlyIfSuccessful: true

                    stash includes: 'AssemblyInfo.vb', name: 'AssemblyInfo.vb'
                }
            }
        }

        stage('increase build version VB.NET linux') {
            agent { node { label 'linux' } }
            when { expression { !test_committer('jenkins')  } }
            steps {
                script {
                    // incremento della versione del file dell'assembly generato
                    env.NEW_VERSION = projedit.increase_version("vbframework", "AssemblyInfo.vb")
                    // aggiunge la versione alla descrizione della compilazione
                    currentBuild.description = currentBuild.description + " version ${env.NEW_VERSION}"

                    archiveArtifacts artifacts: "AssemblyInfo.vb", fingerprint: true, onlyIfSuccessful: true

                    stash includes: 'AssemblyInfo.vb', name: 'AssemblyInfo.vb'
                }
            }
        }
        
        stage('push changes') {
            when { expression { !test_committer('jenkins')  } }
            steps {
                script {
                    unstash 'AssemblyInfo.vb'
                    unstash 'AssemblyInfo.cs'
                    unstash 'write_build_info_l'
                    unstash 'write_build_info_w'

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

        stage('generate version number') {
            when { expression { !test_committer('jenkins')  } }
            steps {
                script {
                    env.NEW_VERSION = (new Date()).format("yy.MM.dd.${env.BUILD_ID}")
                    currentBuild.description = currentBuild.description + " version ${env.NEW_VERSION}"
                }
            }
        }

        stage('set build version C# windows') {
            agent { node { label 'windows' } }
            when { expression { !test_committer('jenkins')  } }
            steps {
                script {
                    // incremento della versione del file dell'assembly generato
                    env.NEW_VERSION = projedit.set_version("csframework", "AssemblyInfo.cs", env.NEW_VERSION)
                    // aggiunge la versione alla descrizione della compilazione
                    currentBuild.description = currentBuild.description + " version ${env.NEW_VERSION}"

                    archiveArtifacts artifacts: "AssemblyInfo.cs", fingerprint: true, onlyIfSuccessful: true

                    stash includes: 'AssemblyInfo.cs', name: 'AssemblyInfo.cs'
                }
            }
        }

        stage('set build version C# linux') {
            agent { node { label 'linux' } }
            when { expression { !test_committer('jenkins')  } }
            steps {
                script {
                    // incremento della versione del file dell'assembly generato
                    env.NEW_VERSION = projedit.set_version("csframework", "AssemblyInfo.cs", env.NEW_VERSION)
                    // aggiunge la versione alla descrizione della compilazione
                    currentBuild.description = currentBuild.description + " version ${env.NEW_VERSION}"

                    archiveArtifacts artifacts: "AssemblyInfo.cs", fingerprint: true, onlyIfSuccessful: true

                    stash includes: 'AssemblyInfo.cs', name: 'AssemblyInfo.cs'
                }
            }
        }

        stage('set build version VB.NET windows') {
            agent { node { label 'windows' } }
            when { expression { !test_committer('jenkins')  } }
            steps {
                script {
                    // incremento della versione del file dell'assembly generato
                    env.NEW_VERSION = projedit.set_version("vbframework", "AssemblyInfo.vb", env.NEW_VERSION)
                    // aggiunge la versione alla descrizione della compilazione
                    currentBuild.description = currentBuild.description + " version ${env.NEW_VERSION}"

                    archiveArtifacts artifacts: "AssemblyInfo.vb", fingerprint: true, onlyIfSuccessful: true

                    stash includes: 'AssemblyInfo.vb', name: 'AssemblyInfo.vb'
                }
            }
        }

        stage('set build version VB.NET linux') {
            agent { node { label 'linux' } }
            when { expression { !test_committer('jenkins')  } }
            steps {
                script {
                    // incremento della versione del file dell'assembly generato
                    env.NEW_VERSION = projedit.set_version("vbframework", "AssemblyInfo.vb")
                    // aggiunge la versione alla descrizione della compilazione
                    currentBuild.description = currentBuild.description + " version ${env.NEW_VERSION}"

                    archiveArtifacts artifacts: "AssemblyInfo.vb", fingerprint: true, onlyIfSuccessful: true

                    stash includes: 'AssemblyInfo.vb', name: 'AssemblyInfo.vb'
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

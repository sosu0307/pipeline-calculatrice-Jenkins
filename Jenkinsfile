pipeline {
    agent none 
    stages {
        stage('Build') {
            agent {
                docker { image 'python:3.8-alpine3.16' } 
            }
            steps {
                sh 'python3.8 -m py_compile sources/prog.py sources/calc.py'
                stash(name: 'compiled-results', includes: 'sources/*.py*')
            }
        }
    }
}

        stage('Test') {
                    agent {
                        docker { image 'grihabor/pytest' }
                    }
                    steps {
                        sh 'pytest -v --junit-xml test-reports/results.xml sources/test_calc.py'
                    }
                    post {
                        always {
                            junit "test-reports/results.xml" // Verarbeitet Testergebnisse grafisch
                        }
                    }
                }
        stage('Deliver') {
            agent any // Läuft auf dem Jenkins-Hauptknoten, um Docker-Befehle zu senden
            environment {
                // Wir definieren Pfade für den Container
                VOLUME = "$(pwd)/sources:/src"
                IMAGE = 'cdrx/pyinstaller-linux'
            }
            steps {
                dir(path: env.BUILD_ID) {
                    unstash(name: 'compiled-results')
                    // Dieser Befehl baut die ausführbare Datei
                    sh "docker run --rm -v ${VOLUME} ${IMAGE} 'pyinstaller -F prog.py'"
                }
            }
            post {
                success {
                    // Das wichtigste: Jenkins speichert die fertige Datei ("Artifact")
                    archiveArtifacts "${env.BUILD_ID}/sources/dist/prog"
                    // Aufräumen: Löscht temporäre Build-Dateien
                    sh "rm -rf ${env.BUILD_ID}/sources/build ${env.BUILD_ID}/sources/dist"
                }
            }
        }

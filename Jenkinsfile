pipeline {
    agent {
        docker {
            image 'python:3.11'  // Gunakan image Python yang sudah memiliki pip
        }
    }
    options {
        timeout(time: 30, unit: 'MINUTES')  // Timeout agar pipeline tidak berjalan terus-menerus
    }
    triggers {
        pollSCM('H/2 * * * *')  // Poll repository setiap 2 menit
    }
    stages {
        stage('Checkout') {
            steps {
                echo "Mengambil kode terbaru dari repository Git"
                checkout scm
            }
        }

        stage('Install Dependencies') {
            steps {
                echo "Menginstal pytest dan pyinstaller tanpa virtual environment"
                sh '''
                    python -m pip install --upgrade pip
                    python -m pip install pytest pyinstaller
                '''
            }
        }

        stage('Build') {
            steps {
                echo "Menjalankan py_compile menggunakan Python langsung"
                sh '''
                    python -m py_compile sources/add2vals.py sources/calc.py
                '''

                echo "Stashing hasil compile"
                stash name: 'compiled-results', includes: 'sources/*.py*'
            }
        }

        stage('Test') {
            steps {
                echo "Menjalankan pytest menggunakan Python langsung"
                sh '''
                    python -m pytest --junit-xml test-reports/results.xml sources/test_calc.py
                '''

                echo "Menyimpan hasil uji"
                junit 'test-reports/results.xml'
            }
        }

        stage('Manual Approval') {
            steps {
                script {
                    echo "Menunggu persetujuan untuk tahap Deploy"
                    def userInput = input(
                        message: 'Lanjutkan ke tahap Deploy?',
                        parameters: [
                            choice(name: 'Continue', choices: ['Proceed', 'Abort'], description: 'Pilih untuk melanjutkan atau menghentikan pipeline')
                        ]
                    )

                    if (userInput == 'Abort') {
                        error('Pipeline dihentikan oleh pengguna.')
                    }
                }
            }
        }

        stage('Deploy') {
            steps {
                echo "Menjalankan aplikasi"
                sh '''
                    nohup python sources/app.py &
                '''

                echo "Menunggu selama 1 menit agar aplikasi tetap berjalan"
                sh 'sleep 60'

                echo "Deploy selesai, aplikasi akan dihentikan."
            }
        }
    }
    post {
        failure {
            echo "Build failed. Mohon periksa log error."
        }
    }
}

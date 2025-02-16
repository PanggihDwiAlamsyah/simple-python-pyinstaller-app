node { 
    properties([
        pipelineTriggers([
            pollSCM('H/2 * * * *')  // Poll setiap 2 menit
        ])
    ])

    try {
        stage('Checkout') {
            echo "Mengambil kode terbaru dari repository Git"
            checkout scm
        }
        
        stage('Install Dependencies') {
            echo "Menggunakan Python yang sudah terinstal di container"

            echo "Menginstal pytest dan pyinstaller tanpa virtual environment"
            sh '''#!/bin/bash
                python3.11 -m pip install --upgrade pip
                python3.11 -m pip install pytest pyinstaller
            '''
        }

        stage('Build') {
            echo "Menjalankan py_compile menggunakan Python langsung"
            sh '''#!/bin/bash
                python3.11 -m py_compile sources/add2vals.py sources/calc.py
            '''

            echo "Stashing hasil compile"
            stash name: 'compiled-results', includes: 'sources/*.py*'
        }

        stage('Test') {
            echo "Menjalankan pytest menggunakan Python langsung"
            sh '''#!/bin/bash
                python3.11 -m pytest --junit-xml test-reports/results.xml sources/test_calc.py
            '''

            echo "Menyimpan hasil uji"
            junit 'test-reports/results.xml'
        }

        stage('Manual Approval') {
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

        stage('Deploy') {
            echo "Menjalankan aplikasi"
            sh '''#!/bin/bash
                nohup python3.11 sources/app.py &
            '''

            echo "Menunggu selama 1 menit agar aplikasi tetap berjalan"
            sh 'sleep 60'

            echo "Deploy selesai, aplikasi akan dihentikan."
        }

    } catch (Exception err) {
        echo "Build failed: ${err}"
        currentBuild.result = 'FAILURE'
    }
}

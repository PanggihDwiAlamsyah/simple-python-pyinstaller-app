node { 
    properties([
        pipelineTriggers([
            pollSCM('H/2 * * * *')  // Poll setiap 2 menit
        ])
    ])

    def venvPath = 'venv'

    try {
        stage('Checkout') {
            echo "Mengambil kode terbaru dari repository Git"
            checkout scm
        }

        stage('Install Dependencies') {
            echo "Menginstal dependencies sistem yang dibutuhkan"
            sh '''
                sudo apt update
                sudo apt install -y python3.11 python3.11-dev python3.11-venv python3-pip build-essential
            '''

            echo "Membuat ulang virtual environment"
            sh 'rm -rf venv && python3 -m venv venv'

            echo "Mengaktifkan virtual environment dan menginstal pytest serta pyinstaller"
            sh "source ${venvPath}/bin/activate && pip install --upgrade pip && pip install pytest pyinstaller"
        }

        stage('Build') {
            echo "Mengaktifkan virtual environment dan menjalankan py_compile"
            sh "source ${venvPath}/bin/activate && python -m py_compile sources/add2vals.py sources/calc.py"

            echo "Stashing hasil compile"
            stash name: 'compiled-results', includes: 'sources/*.py*'
        }

        stage('Test') {
            echo "Mengaktifkan virtual environment dan menjalankan pytest"
            sh "source ${venvPath}/bin/activate && pytest --junit-xml test-reports/results.xml sources/test_calc.py"

            echo "Menyimpan hasil uji"
            junit 'test-reports/results.xml'
        }

        stage('Deliver') {
            echo "Mengaktifkan virtual environment dan menjalankan pyinstaller"
            sh "source ${venvPath}/bin/activate && which python"
            sh "source ${venvPath}/bin/activate && python --version"

            echo "Menjalankan PyInstaller dengan opsi tambahan"
            sh """
                export LD_LIBRARY_PATH=/usr/lib/x86_64-linux-gnu:$LD_LIBRARY_PATH
                source ${venvPath}/bin/activate && pyinstaller --onefile sources/add2vals.py --log-level=DEBUG
            """

            echo "Mengarsipkan hasil build"
            archiveArtifacts artifacts: 'dist/add2vals', fingerprint: true
        }
    } catch (Exception err) {
        echo "Build failed: ${err}"
        currentBuild.result = 'FAILURE'
    }
}

node {
    // Menetapkan properti pipeline untuk polling SCM setiap 2 menit
    properties([
        pipelineTriggers([
            pollSCM('H/2 * * * *')  // Poll setiap 2 menit
        ])
    ])

    // Definisi variabel untuk jalur virtual environment
    def venvPath = 'venv/bin/activate'

    try {
        stage('Checkout') {
            echo "Mengambil kode terbaru dari repository Git"
            checkout scm
        }

        stage('Install Dependencies') {
            echo "Membuat virtual environment"
            sh 'python3 -m venv venv'
            
            echo "Mengaktifkan virtual environment dan menginstal pytest serta pyinstaller"
            sh ". ${venvPath} && pip install --upgrade pip && pip install pytest pyinstaller"
        }

        stage('Build') {
            echo "Mengaktifkan virtual environment dan menjalankan py_compile"
            sh ". ${venvPath} && python -m py_compile sources/add2vals.py sources/calc.py"
            
            echo "Stashing hasil compile"
            stash name: 'compiled-results', includes: 'sources/*.py*'
        }

        stage('Test') {
            echo "Mengaktifkan virtual environment dan menjalankan pytest"
            sh ". ${venvPath} && pytest --junit-xml test-reports/results.xml sources/test_calc.py"
            
            echo "Menyimpan hasil uji"
            junit 'test-reports/results.xml'
        }

        stage('Deliver') {
            echo "Mengaktifkan virtual environment dan menjalankan pyinstaller"
            sh ". ${venvPath} && pyinstaller --onefile sources/add2vals.py"
            
            echo "Mengarsipkan hasil build"
            archiveArtifacts artifacts: 'dist/add2vals', fingerprint: true
        }
    } catch (Exception err) {
        echo "Build failed: ${err}"
        currentBuild.result = 'FAILURE'
    }
}
"Mengaktifkan virtual environment dan menjalankan pyinstaller"
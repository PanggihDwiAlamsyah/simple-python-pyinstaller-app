node {
    // Polling Git repository setiap 2 menit sekali
    properties([
        pollSCM('H/2 * * * *')  // Poll setiap 2 menit
    ])
    
    try {
        stage('Checkout') {
            // Checkout kode dari SCM (Git)
            checkout scm
        }
        
        stage('Install Dependencies') {
            // Menginstal dependensi Python
            sh 'python3 -m venv venv'
            sh '. venv/bin/activate && pip install --upgrade pip && pip install pytest pyinstaller'
        }

        stage('Build') {
            // Mengkompilasi kode Python
            sh '. venv/bin/activate && python -m py_compile sources/add2vals.py sources/calc.py'
            stash name: 'compiled-results', includes: 'sources/*.py*'
        }

        stage('Test') {
            // Menjalankan test dengan pytest
            sh '. venv/bin/activate && pytest --junit-xml test-reports/results.xml sources/test_calc.py'
            junit 'test-reports/results.xml'
        }

        stage('Deliver') {
            // Menyusun aplikasi dengan pyinstaller
            sh '. venv/bin/activate && pyinstaller --onefile sources/add2vals.py'
            archiveArtifacts 'dist/add2vals'
        }

        stage('Commit Changes') {
            // Menggunakan credentials untuk commit dan push ke repository Git
            withCredentials([usernamePassword(credentialsId: 'git-credentials', usernameVariable: 'GIT_USER', passwordVariable: 'GIT_PASS')]) {
                // Mengatur konfigurasi git
                sh 'git config user.name "PanggihDwiAlamsyah"'
                sh 'git config user.email "pdwialamsyah@gmail.com"'
                // Menambahkan perubahan ke git
                sh 'git add .'
                sh 'git commit -m "Update changes from Jenkins build" || echo "No changes to commit"'
                // Melakukan push ke repository Git
                sh 'git push https://$GIT_USER:$GIT_PASS@github.com/yourrepo.git'
            }
        }
    } catch (Exception e) {
        // Menandai build gagal jika terjadi error
        currentBuild.result = 'FAILURE'
        throw e
    } finally {
        // Membersihkan workspace setelah pipeline selesai
        cleanWs()
    }
}
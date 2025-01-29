node {
    properties([pollSCM('H/2 * * * *')])  // Poll SCM setiap 2 menit

    try {
        stage('Install Dependencies') {
            sh 'python3 -m venv venv'
            sh '. venv/bin/activate && pip install --upgrade pip && pip install pytest pyinstaller'
        }

        stage('Build') {
            sh '. venv/bin/activate && python -m py_compile sources/add2vals.py sources/calc.py'
            stash(name: 'compiled-results', includes: 'sources/*.py*')
        }

        stage('Test') {
            sh '. venv/bin/activate && pytest --junit-xml test-reports/results.xml sources/test_calc.py'
            junit 'test-reports/results.xml'
        }

        stage('Deliver') {
            sh '. venv/bin/activate && pyinstaller --onefile sources/add2vals.py'
            archiveArtifacts 'dist/add2vals'
        }
    } catch (Exception e) {
        currentBuild.result = 'FAILURE'
        throw e
    } finally {
        cleanWs()
    }
}

pipeline {
  agent { 
    docker { 
      image 'conda/miniconda3-centos7:latest'
    } 
  }
  stages {
    stage('root-setup') {
      steps {
        sh 'yum update -y'
        sh 'yum install -y epel-release'
        sh 'yum install -y gcc gcc-c++ git gcc-gfortran libboost-dev make wget boost boost-thread boost-devel'
        dir('idaes-dev') {
          git url: 'https://github.com/makaylas/idaes-dev.git',
          credentialsId: '6ca01274-150a-4dd4-96ec-f0d117b0ea95'
        }
      }
    }
    stage('idaes-dev setup') {
      steps {
        sh '''
         cd idaes-dev
         conda create -n idaes python=3.7 pytest
         source activate idaes
         pip install -r requirements-dev.txt --user jenkins
         export TEMP_LC_ALL=$LC_ALL
         export TEMP_LANG=$LANG
         export LC_ALL=en_US.utf-8
         export LANG=en_US.utf-8
         python setup.py install
         export LC_ALL=$TEMP_LC_ALL
         export LANG=$TEMP_LANG
         conda deactivate
         '''
      }
    }
    stage('idaes-ext setup') {
      steps {
        sh '''
         ls
         source activate idaes
         rm -rf coinbrew dist-lib dist-solvers
         bash scripts/compile_solvers.sh
         bash scripts/compile_libs.sh
         cp dist-solvers/idaes-solvers.tar.gz dist-lib/idaes-solvers-linux-64.tar.gz
         mv dist-lib/idaes-lib.tar.gz dist-lib/idaes-lib-linux-64.tar.gz
         idaes get-extensions --url file://`pwd`/dist-lib
         conda deactivate
         '''
      }
    }
    stage('idaes-dev test') {
      steps {
        sh '''
         cd idaes-dev
         source activate idaes
         pylint -E --ignore-patterns="test_.*" idaes || true
         pytest -c pytest.ini idaes
         conda deactivate
         '''
      }   
    }
  }
  post {
    always {
      emailext attachLog: true, body: "${currentBuild.result}: ${BUILD_URL}", replyTo: 'idaes.jenkins@lbl.gov',
       subject: "Build Log: ${JOB_NAME} - Build ${BUILD_NUMBER} ${currentBuild.result}", to: 'idaes.jenkins@lbl.gov'
    }
  }
}

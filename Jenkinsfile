pipeline {
   agent {label "${agent_label}"}
      stages {
         stage('Clone Tensorflow Repository') {
            steps {
             sh '''#!/bin/bash -xe
             export TF_BRANCH="${tf_branch}"
             cd /
             echo 'jenkins' | sudo -S git clone --branch=${TF_BRANCH} --depth=1 https://github.com/tensorflow/tensorflow.git
             cd tensorflow
             echo 'jenkins' |sudo -S git checkout ${TF_BRANCH}
             echo 'jenkins' |sudo -S updatedb
                '''
            }
    }
         stage('Configure Build ENV & Build TensorFlow Package From Sources') {
            steps {
             sh '''#!/bin/bash -xe
                   cd /
                   echo 'jenkins' | sudo -S cp build_tf_package.sh /tensorflow
                   cd tensorflow
                   echo 'jenkins' | sudo -S bash build_tf_package.sh 
                '''
            }
    }
         stage('Install Tensorflow Package') {
            steps {
                  sh '''#!/bin/bash -xe
                  mv /home/jenkins/tensorflow-*.whl $WORKSPACE
                  echo 'jenkins' | sudo -S pip --no-cache-dir install --upgrade $WORKSPACE/tensorflow-*.whl
                  echo 'jenkins' | sudo -S rm -rf /root/.cache
                     '''
            }
     }
         stage('Testing Tensorflow Installation') {
            steps {
             sh '''#!/bin/bash -xe
                   cd /
                   python unitest.py 
                       if [ "$?" != "0" ]; then
                          echo "Tensorflow build Failed!!!"
                          exit -1
                       fi
                 ''' 
            }
    }
         stage('Push Arifact To Network Share') {
            steps {
             sh '''#!/bin/bash -xe
                   export TFLOW=$(cd $WORKSPACE && find -type f -name "tensorflow*.whl" | cut -c 3-)
                   pv $WORKSPACE/${TFLOW} > /media/common/DOCKER_IMAGES/Tensorflow/Current/${TFLOW}
                   cd $WORKSPACE
                   md5sum /media/common/DOCKER_IMAGES/Tensorflow/Current/${TFLOW} > ${TFLOW}.md5
                   md5sum -c ${TFLOW}.md5 
                       if [ "$?" != "0" ]; then
                          echo "SHA1 changed! Security breach? Job Will Be Marked As Failed!!!"
                          exit -1
                       fi
                 ''' 
            }
    }
         stage('Cleanup Build Folders') {
            steps {
             sh '''#!/bin/bash -xe
                   cd $WORKSPACE
                   rm -rf tensorflow-*
                   rm -f /home/jenkins/tensorflow-*
                   echo 'jenkins' | sudo -S rm -rf /whl 
                 ''' 
            }
     }
}      
         post {
            always {
               script {
                  if (currentBuild.result == null) {
                     currentBuild.result = 'SUCCESS' 
                  }
               }
               step([$class: 'Mailer',
                     notifyEveryUnstableBuild: true,
                     recipients: "igor.rabkin@xiaoyi.com",
                     sendToIndividuals: true])
            }
         } 
}

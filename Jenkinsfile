def bucket = 'adweb'
def functionName = 'ldrtest'
def region = 'ap-south-1'

node('master'){
    stage('Checkout'){
        checkout scm
    }

    stage('Test'){

    }

    stage('Build'){
        
        sh "zip ${commitID()}.zip build"
    }

    stage('Push'){

            withAWS(region:'ap-south-1',credentials:'awsCredentials') {
//              s3Delete(bucket: 'adweb', path:'**/*')
              s3Upload(bucket: 'adweb', workingDir:'build', includePathPattern:'**/*');
            }

withCredentials([[$class: 'UsernamePasswordMultiBinding', credentialsId: 'awsCredentials', usernameVariable: 'AWS_ACCESS_KEY_ID', passwordVariable: 'AWS_SECRET_ACCESS_KEY']]) {
//        AWS("--region=ap-south-1 s3 ls")
sh "aws s3 cp ${commitID()}.zip s3://${bucket}"

    }
           withAWS(credentialsId: 'awsCredentials',region: 'ap-south-1') {

              sh "aws s3 cp ${commitID()}.zip s3://${bucket}"   
        }

//        sh "aws s3 cp ${commitID()}.zip s3://${bucket}"
    }

    stage('Deploy'){
        sh "aws lambda update-function-code --function-name ${functionName} \
                --s3-bucket ${bucket} \
                --s3-key ${commitID()}.zip \
                --region ${region}"
    }

    if (env.BRANCH_NAME == 'master') {
        stage('Publish') {
            sh "aws lambda publish-version --function-name ${functionName} \
                    --region ${region}"
        }
    }
}

def commitID() {
    sh 'git rev-parse HEAD > .git/commitID'
    def commitID = readFile('.git/commitID').trim()
    sh 'rm .git/commitID'
    commitID
}

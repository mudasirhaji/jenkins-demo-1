node {

 timestamps {

  def scmVars
  def commit
  def author
  def error_message
  def stage_title

  try {


   echo "Running ${env.BUILD_DISPLAY_NAME} on ${env.JOB_NAME} : ${env.BRANCH_NAME} ..."

   stage('Get Source Code') {

    scmVars = checkout scm
    commit = sh(returnStdout: true, script: 'git log -1 --pretty=%B | cat')
    author = sh(returnStdout: true, script: 'git log -1 --pretty=%an | cat')

    if (!env.BRANCH_NAME.startsWith('PR')) {
     sh "git checkout ${env.BRANCH_NAME}"
    }

   }


   stage("Check Errors") {

    try {
     echo "Check Syntax Error.."
     sh 'find app -name "*.php" -print0 | xargs -0 -n1 php -l'

    } catch (err) {
     stage_title = "Check Errors"
     error_message = err.getMessage()
     throw err
    }

   }


  stage("Code Style Check") {

    try {
     echo "Check Code Style to have a standard code.."
    } catch (err) {
     stage_title = "Code Style Check"
     error_message = err.getMessage()
     throw err
    }

   }

  stage('SonarQube') {

    try {
    echo "Scan Sonar..."

    } catch (err) {
     stage_title = "SonarQube"
     error_message = err.getMessage()
     throw err
    }

   }


  stage('Create Docker') {

    try {
      echo "Create Docker..."
      sh 'docker build .'
    } catch (err) {
     stage_title = "SonarQube"
     error_message = err.getMessage()
     throw err
    }

   }


  stage('Uplaod Package') {

    try {
      echo "Copy composer.json  to Server..."
      sh 'scp composer.json mudasir@34.243.222.144'
    } catch (err) {
     stage_title = "SonarQube"
     error_message = err.getMessage()
     throw err
    }

   }

   def deploy_commands = "php artisan view:clear && php artisan config:clear && php artisan cache:clear && php artisan route:clear"
   
   if (env.BRANCH_NAME == 'develop' ) {
   

      stage('Deploy Dev Server') {

        remotes = ['dev_srv_1'];
        remotes.each {
            remote ->

            try {
                sh "git remote add dev_${remote} ${remote}:/var/www/repos/jenkins-demo.git"
            } catch (def err) {
                echo err.getMessage();
            }

            sh "git push dev_${remote} ${env.BRANCH_NAME}"

            sh "ssh ${remote} \"cd /var/www/sites/jenkins-demo.dev/ && ${deploy_commands}\""

        }

      }

      stage('Deploy Test Server') {

            remotes = ['test_srv_1'];
            remotes.each {
            remote ->

            try {
                sh "git remote add test_${remote} ${remote}:/var/www/repos/jenkins-demo.git"
            } catch (def err) {
                echo err.getMessage();
            }

            sh "git push test_${remote} ${env.BRANCH_NAME}"

            sh "ssh ${remote} \"cd /var/www/sites/jenkins-demo.test/ && ${deploy_commands}\""


            }

      }

   }


     if (env.BRANCH_NAME == 'main' ) {
   

      stage('Deploy Pro Servers') {

        echo "Deploying to Pro Serves...."
        // remotes = ['pro_srv_1'];
        // remotes.each {
        //     remote ->

        //     try {
        //         sh "git remote add pro_${remote} ${remote}:/var/www/repos/jenkins-demo.git"
        //     } catch (def err) {
        //         echo err.getMessage();
        //     }

        //     sh "git push pro_${remote} ${env.BRANCH_NAME}"

        //     sh "ssh ${remote} \"cd /var/www/sites/jenkins-demo.pro/ && ${deploy_commands}\""

        // }

      }



   }



   stage('Validation') {
    stage_title = "Validation"

    try {

     def response = sh(
      returnStdout: true,
      script: "curl -Is https://google.com | head -n 1"
     );

     if (!response.contains('200 OK')) {
      error_message = "Dev Server is down"
     } else {
        echo "Dev Server is OK!"
     }

    } catch (def err) {
     error_message = err.getMessage()
    }


    try {

     def response = sh(
      returnStdout: true,
      script: "curl -Is https://google.com | head -n 1"
     );

     if (!response.contains('200 OK')) {
      error_message = "Test Server is down"
     } else {
        echo "Test Server is OK!"
     }

    } catch (def err) {
     error_message = err.getMessage()
    }
   }


  } catch (def err) {


   error_message = err.getMessage()


   throw err

  } finally {

   if (error_message) {

    def error_attachments = [
     [
      pretext: ":x: *${env.JOB_NAME}* FAILED after ${currentBuild.durationString} @channel",
      color: 'danger',
      title: currentBuild.fullDisplayName,
      title_link: "${env.BUILD_URL}console",
      fallback: "${env.JOB_NAME} FAILED after ${currentBuild.durationString}",
      fields: [
       [
        title: "Branch",
        value: env.BRANCH_NAME,
        short: false
       ],
       [
        title: "Build No",
        value: env.BUILD_DISPLAY_NAME,
        short: false
       ],
       [
        title: "Error",
        value: error_message,
        short: false
       ],
       [
        title: "Stage",
        value: stage_title,
        short: false
       ],
       [
        title: "Commit",
        value: commit,
        short: false
       ],
       [
        title: "Author",
        value: author,
        short: false
       ]
      ],
      actions: [
       [
        type: "button",
        text: "Details",
        url: "${env.JENKINS_URL}blue/organizations/jenkins/jenkins-demo/detail/${env.BRANCH_NAME}/${BUILD_NUMBER}/pipeline"
       ]
      ]
     ]
    ]

    //slackSend(channel: 'jenkins', attachments: error_attachments)
   } else {

    def attachments = [
     [
      pretext: ":tada: *${env.JOB_NAME}* ${currentBuild.currentResult} after ${currentBuild.durationString}",
      color: 'good',
      title: currentBuild.fullDisplayName,
      title_link: "${env.BUILD_URL}console",
      fallback: "${env.JOB_NAME} ${currentBuild.currentResult} after ${currentBuild.durationString}",
      fields: [
       [
        title: "Branch",
        value: env.BRANCH_NAME,
        short: false
       ],
       [
        title: "Build No",
        value: env.BUILD_DISPLAY_NAME,
        short: false
       ],
       [
        title: "Commit",
        value: commit,
        short: false
       ],
       [
        title: "Author",
        value: author,
        short: false
       ],
      ],
      actions: [
       [
        type: "button",
        text: "Details",
        url: "${env.JENKINS_URL}blue/organizations/jenkins/jenkins-demo/detail/${env.BRANCH_NAME}/${BUILD_NUMBER}/pipeline"
       ]
      ]

     ]
    ]

    //slackSend(channel: 'jenkins', attachments: attachments)
   }

   cleanWs();

  }
 }

}

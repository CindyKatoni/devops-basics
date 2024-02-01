pipeline{
    tools{
        jdk 'myjava'
        maven 'mymaven'
    }
	agent any
      stages{
           stage('Checkout'){
              steps{
		 echo 'cloning..'
                 git 'https://github.com/theitern/DevopsBasics.git'
              }
          }
          stage('Compile'){
              steps{
                  echo 'compiling..'
                  sh 'mvn compile'
	      }
          }
          stage('CodeReview'){
              steps{
		    
		  echo 'codeReview'
                  sh 'mvn pmd:pmd'
              }
          }
        //    stage('UnitTest'){
        //       steps{
	    //      echo 'Testing'
        //           sh 'mvn test'
        //       }
        //        post {
        //        success {
        //            junit 'target/surefire-reports/*.xml'
        //        }
        //    }	
        //   }
          stage('Package'){
              steps{
                  sh 'mvn package'
              }
          }
      }
}


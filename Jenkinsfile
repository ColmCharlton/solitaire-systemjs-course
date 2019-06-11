stage 'CI'
node {

	checkout scm
	
    //git branch: 'jenkins2-course', 
    //    url: 'https://github.com/ColmCharlton/solitaire-systemjs-course'
		

    // pull dependencies from npm
    bat 'npm install'
 

    // stash code & dependencies to expedite subsequent testing
    // and ensure same code & dependencies are used throughout the pipeline
    // stash is a temporary archive
    stash name: 'everything', 
          excludes: 'test-results/**', 
          includes: '**'
    
    // test with PhantomJS for "fast" "generic" results
   bat 'npm run test-single-run -- --browsers PhantomJS'
    
    
    // archive karma test results (karma is configured to export junit xml files)
    step([$class: 'JUnitResultArchiver', 
          testResults: 'test-results/**/test-results.xml'])
          
}


//Second agent, can be on any computer or a server
node('win') {
    bat 'ls'
    bat 'rm -rf'
    unstash 'everything' 
    bat 'ls'
    //bat 'npm run test-single-run -- --browsers Chrome'
}

//Parallel integration testing
stage 'Browser Testing'
parallel chrome: {
    runTests("Chrome")
    }, firefox:{
    runTests("Firefox")
    }
    
def runTests(browser) {
    node {
        bat 'rm -rf *' //Clean workspace
        unstash 'everything'
        bat "npm run test-single-run -- --browsers ${browser}"
        step([$class: 'JUnitResultArchiver',
                testResults: 'test-results/**/test-results.xml'])
        
    }
}

//Must to wrapped in node
node {
    notify("Deploy to staging?")
}
    
//Do not wrap in a node, it will cause the pipeline to pause
//Use flyweight node
input 'Deploy to staging?'


// limit concurrency so we don't perform simultaneous deploys
// and if multiple pipelines are executing, 
// newest is only that will be allowed through, rest will be canceled
// stage name: 'Deploy to staging', concurrency: 1
// node {
//     // write build number to index page so we can see this update
//     // on windows use: bat "echo '<h1>${env.BUILD_DISPLAY_NAME}</h1>' >> app/index.html"
//     bat "echo '<h1>${env.BUILD_DISPLAY_NAME}</h1>' >> app/index.html"
    
//     // deploy to a docker container mapped to port 3000
//     // on windows use: bat 'docker-compose up -d --build'
//     bat 'docker-compose up -d --build'
    
//     notify 'Solitaire Deployed!'
// }

def notify(status){
    emailext (
      to: "wesmdemos@gmail.com",
      subject: "${status}: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]'",
      body: """<p>${status}: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]':</p>
        <p>Check console output at <a href='${env.BUILD_URL}'>${env.JOB_NAME} [${env.BUILD_NUMBER}]</a></p>""",
    )
}
stage 'CI'
node {

    checkout scm
    
    //git branch: 'jenkins2-course', 
    //    url: 'https://github.com/g0t4/solitaire-systemjs-course'

    // pull dependencies from npm
    // on windows use: bat 'npm install'
    //sh 'npm install'
    bat 'npm install'

    // stash code & dependencies to expedite subsequent testing
    // and ensure same code & dependencies are used throughout the pipeline
    // stash is a temporary archive
    stash name: 'everything', 
          excludes: 'test-results/**', 
          includes: '**'
    
    // test with PhantomJS for "fast" "generic" results
    // on windows use: bat 'npm run test-single-run -- --browsers PhantomJS'
    //sh 'npm run test-single-run -- --browsers PhantomJS'
    bat 'npm run test-single-run -- --browsers PhantomJS'
    
    // archive karma test results (karma is configured to export junit xml files)
    step([$class: 'JUnitResultArchiver', 
          testResults: 'test-results/**/test-results.xml'])
          
}

node ('Windows') {
    bat 'dir'
    // on windows use: bat 'del /S /Q *'
    bat 'del /S /Q *'

    unstash 'everything'

    // on windows use: 
    bat 'dir'
}

// parallel intergration testing
stage 'Browser Testing'
parallel chrome: {
    runTests("Chrome")
}, firefox: {
    runTests("Firefox")
}

def runTests(browser) {
    node {
        bat 'del /S /Q *'
        unstash 'everything'
        bat "npm run test-single-run -- --browsers ${browser}"
        // archive karma test results (karma is configured to export junit xml files)
        step([$class: 'JUnitResultArchiver', 
                testResults: 'test-results/**/test-results.xml'])
    }
}

node {
    notify("Deploy to staging?")
}

input 'Deploy to staging?'


def notify(status){
    emailext (
      to: "fengboyuannj@gmail.com",
      subject: "${status}: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]'",
      body: """<p>${status}: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]':</p>
        <p>Check console output at <a href='${env.BUILD_URL}'>${env.JOB_NAME} [${env.BUILD_NUMBER}]</a></p>""",
    )
}
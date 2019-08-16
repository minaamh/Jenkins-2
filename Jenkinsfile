stage 'CI'
node {

		
		git branch: 'jenkins2-course', 
        url: 'https://github.com/g0t4/solitaire-systemjs-course'

    // pull dependencies from npm
    // on windows use: bat 'npm install'
    bat label: '', script: 'npm install'

    // stash code & dependencies to expedite subsequent testing
    // and ensure same code & dependencies are used throughout the pipeline
    // stash is a temporary archive
    stash name: 'everything', 
          excludes: 'test-results/**', 
          includes: '**'
    
}

node('windows'){
    bat label: '', script: '''dir 
    del *.* /F /Q'''
    bat label: '', script: 'dir'
}

//parallel integration testing
stage 'Browser Testing'
parallel chrome: {
       runTests("Chrome")
    }, firefox: {
        runTests("Firefox")
    }, phantomjs: {
        runTests("PhantomJS")
    }

def runTests(browser){
    node{
        bat label: '', script: '''del *.* /F /Q'''
        unstash 'everything'
        bat label: '', script: "npm run test-single-run -- --browsers ${browser}"
        // archive karma test results (karma is configured to export junit xml files)
        step([$class: 'JUnitResultArchiver', testResults: 'test-results/**/test-results.xml'])
    }
}



def notify(status){
    emailext (
      to: "mina.abdelsayed@coastcapitalsavings.com",
      subject: "${status}: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]'",
      body: """<p>${status}: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]':</p>
        <p>Check console output at <a href='${env.BUILD_URL}'>${env.JOB_NAME} [${env.BUILD_NUMBER}]</a></p>""",
    )
}

node {
    notify ("Deploy to staging?")
}
input 'Deploy to staging?'

stage name:'Deploy', concurrency:1

node{
    bat label: '', script: "echo '<h1>${env.BUILD_DISPLAY_NAME}</h1>'>>app/index.html"
}
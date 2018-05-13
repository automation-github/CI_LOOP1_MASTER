#!groovy

def target_cluster = "10.65.173.4"

def BuildJob(jobName) {
    timeout(time: 180, unit: 'MINUTES') {   
      try {
        stage(jobName) {    
            def res = build job:jobName, propagate: false
            def ws_name = jobName.split("/")[0]
            // sh "cp -p /var/lib/jenkins/workspace/$ws_name/*.xml /var/lib/jenkins/workspace/CI_LOOP1_MASTER"
            result = res.result

            if (result.equals("SUCCESS")) {
            } else {
               error 'FAIL' // this fails the stage
            }
         }
      } catch (e) {
          currentBuild.result = 'UNSTABLE'
          result = "FAIL" // make sure other exceptions are recorded as failure too
      }
    }
}


node {
//  properties([
//    pipelineTriggers([
//        triggers: [
//            [
//                $class: 'jenkins.triggers.ReverseBuildTrigger',
//                upstreamProjects: "git_pull", threshold: hudson.model.Result.SUCCESS
//            ]
//        ]
//    ]),
//        ])
    
  dir('/var/lib/jenkins/workspace/CI_LOOP1_MASTER') {
      stage('CI_LOOP1_MASTER') {
        parallel (
        // insert tests here
          'test_npvr_ingest_ci1_docker':
          {
            BuildJob('test_npvr_ingest_ci1_docker')
          },
          ' test_rsdvr_ingest_ci1_docker':  
          {
            BuildJob('  test_rsdvr_ingest_ci1_docker')
          }
        )
      }

    stage('copy xmls') {
      sh "scp -p root@$target_cluster:/root/auto_tools/results/5.1/test_*.xml ."
    }

      stage('publish junit results') {
        junit(testResults: '*.xml', healthScaleFactor: 1.0, allowEmptyResults: true)
      }
    }
}
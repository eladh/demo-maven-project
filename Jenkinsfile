node {
    def server
    def buildInfo
    def rtMaven
    def scanConfig

    stage ('Build') {
        git url: 'https://github.com/eladh/demo-maven-project.git'
    }

    stage ('Artifactory configuration') {
        server = Artifactory.server SERVER_ID

        rtMaven = Artifactory.newMavenBuild()
        rtMaven.tool = "maven" // Tool name from Jenkins configuration
        rtMaven.deployer releaseRepo: 'libs-release-local', snapshotRepo: 'libs-snapshot-local', server: server
        rtMaven.resolver releaseRepo: 'libs-release', snapshotRepo: 'libs-snapshot', server: server
        rtMaven.deployer.deployArtifacts = false // Disable artifacts deployment during Maven run

        buildInfo = Artifactory.newBuildInfo()

        scanConfig = [
                'buildName'      : buildInfo.name,
                'buildNumber'    : buildInfo.number,
                'failBuild'      : false
              ]
    }

    stage ('Test') {
        rtMaven.run pom: 'pom.xml', goals: 'clean test'
    }

    stage ('Install') {
        rtMaven.run pom: 'pom.xml', goals: 'install', buildInfo: buildInfo
    }



    stage ('Deploy') {
        rtMaven.deployer.deployArtifacts buildInfo
    }

    stage ('Publish build info') {
        server.publishBuildInfo buildInfo
    }

     stage('Xray Scan') {
          def scanResult = server.xrayScan scanConfig
          print scanResult
          if(FAIL_ON_XRAY.toBoolean() && scanResult.foundVulnerable) {
            error("Build failed because ${scanResult.scanMassege}")
        }
      }


}

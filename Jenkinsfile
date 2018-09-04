node {
	properties([
		[$class: 'BuildDiscarderProperty', strategy: [$class: 'LogRotator', numToKeepStr: '15']],
		parameters([
			choice(choices: 'oxygen\nphoton\nlatest', description: 'Which Target Platform should be used?', name: 'target_platform')
		])
	])
	
	stage('Checkout') {
		checkout scm
		if ("latest" == params.target_platform) {
			currentBuild.displayName = "#${BUILD_NUMBER}(x)"
		} else if ("photon" == params.target_platform) {
			currentBuild.displayName = "#${BUILD_NUMBER}(p)"
		} else {
			currentBuild.displayName = "#${BUILD_NUMBER}(o)"
		}
	}
	
	stage('Gradle Build') {
		sh "./gradlew clean build createLocalMavenRepo -PuseJenkinsSnapshots=true -x test -PignoreTestFailures=true --refresh-dependencies --continue"
	}
	
	def workspace = pwd()
	def mvnHome = tool 'M3'
	env.M2_HOME = "${mvnHome}"
	env.MAVEN_HOME = "${mvnHome}"
	dir('.m2/repository/org/eclipse/xtext') { deleteDir() }
	dir('.m2/repository/org/eclipse/xtend') { deleteDir() }
	
	stage('Maven Plugin Build') {
		sh "${mvnHome}/bin/mvn -f maven-pom.xml --batch-mode --update-snapshots -fae -PuseJenkinsSnapshots -Dmaven.test.failure.ignore=true -Dmaven.repo.local=${workspace}/.m2/repository -Dxtext.workspace.dir=${workspace} -Darchetype.test.localRepositoryPath=${workspace}/.m2/repository -Dorg.slf4j.simpleLogger.log.org.apache.maven.cli.transfer.Slf4jMavenTransferListener=warn clean deploy"
	}
	
	
	archive 'build/**, **/target/work/data/.metadata/.log, **/hs_err_pid*.log'
}

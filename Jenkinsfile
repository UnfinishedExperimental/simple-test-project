def wsRepo = '.m2repo'
def ops = "-B -Dmaven.repo.local=$wsRepo"

stage("checkout") {
    node {
        git "https://github.com/UnfinishedExperimental/simple-test-project.git"
        stash "sources"
    }
}


stage("aaa") {
    node {
        unstash "sources"
        def output
        docker.image('maven:alpine').inside {
            sh "mvn build-helper:remove-project-artifact $ops"
            output = sh returnStdout: true, script: "mvn clean install $ops -pl aaa -am"
        }
        echo output
        stash includes: stashList(output, wsRepo), name: 'aaa-artifacts'
    }
}

parallel b: {

    stage("bbb") {
        node {
            unstash "sources"
            docker.image('maven:alpine').inside {
                sh returnStdout: true, script: "mvn build-helper:remove-project-artifact $ops"
                unstash "aaa-artifacts"
                sh "mvn clean install $ops -pl bbb"
            }
        }
    }
}, c: {

    stage("ccc") {
        node {
            unstash "sources"
            docker.image('maven:alpine').inside {
                sh returnStdout: true, script: "mvn build-helper:remove-project-artifact $ops"
                unstash "aaa-artifacts"
                sh "mvn clean install $ops -pl ccc"
            }
        }
    }
}


def stashList(String output, String repo) {
    def installs = output.split('\n').findAll { it -> it.startsWith("[INFO] Installing") && !it.contains(" at end") && !it.contains("Installing artifact ") }
    def artifacts = installs.collect { it -> it.split(" ")[4] }
    artifacts.collect { l -> l.substring(l.indexOf(repo)) }.join(",")
}

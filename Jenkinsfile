def wsRepo = '.m2repo'
def ops = "-B -Dmaven.repo.local=$wsRepo"

stage("checkout") {
    node {
        git "https://github.com/UnfinishedExperimental/simple-test-project.git"
        stash "sources"
    }
}

buildAndStash("aaa", true)

parallel b: {
    buildAndStash(project: "bbb", deps: ["aaa"] )
}, c: {
    buildAndStash(project:"ccc", deps: ["aaa"])
}

def buildAndStash(String project, boolean alsoMake = false, String[] deps = []) {
    stage(project) {
        node {
            unstash "sources"
            def output
            docker.image('maven:alpine').inside {
                sh "mvn build-helper:remove-project-artifact $ops"
                for (String d : deps) {
                    unstash "$d-artifacts"
                }
                output = sh returnStdout: true, script: "mvn clean install $ops -pl $project ${alsoMake ? '-am' : ''}"
            }
            echo output
            stash includes: stashList(output, wsRepo), name: "$project-artifacts"
        }
    }
}

String stashList(String output, String repo) {
    def installs = output.split('\n').findAll { it -> it.startsWith("[INFO] Installing") && !it.contains(" at end") && !it.contains("Installing artifact ") }
    def artifacts = installs.collect { it -> it.split(" ")[4] }
    artifacts.collect { l -> l.substring(l.indexOf(repo)) }.join(",")
}

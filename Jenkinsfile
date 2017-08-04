stage("checkout") {
    node {
        git "https://github.com/UnfinishedExperimental/simple-test-project.git"
        stash "sources"
    }
}

buildAndStash("aaa", alsoMake: true)

parallel b: {
    buildAndStash("bbb", deps: ["aaa"])
}, c: {
    buildAndStash("ccc", deps: ["aaa"])
}

class C {
    static final wsRepo = '.m2repo'
    static final ops = "-B -Dmaven.repo.local=$wsRepo"
}

/**
 * can be called like this:
 * buildAndStash('my-project', deps: ['foo', 'bar'], alsoMake: true)
 *
 * @param map for optional parameters deps = [] & alsoMake = false
 * @param project maven project to build
 * @return
 */
def buildAndStash(Map map = [:], String project) {
    //convert to String array, because otherwise deps would be a list. And calling the length method on it is forbidden
    //by Jenkins security sandbox
    def deps = (map.deps ?: []) as String[]
    def alsoMake = map.alsoMake ?: false

    stage(project) {
        node {
            unstash "sources"
            def output
            docker.image('maven:alpine').inside {
                sh "mvn build-helper:remove-project-artifact ${C.ops}"
                //can not use foreach loop, because it does not work with Pipeline CPS Transform see [JENKINS-27421]
                for (int i = 0; i < deps.length; i++) {
                    def d = deps[i]
                    unstash "$d-artifacts"
                }
                output = sh returnStdout: true, script: "mvn clean install ${C.ops} -pl $project ${alsoMake ? '-am' : ''}"
            }
            echo output
            stash includes: stashList(output, C.wsRepo), name: "$project-artifacts"
        }
    }
}

String stashList(String output, String repo) {
    def lines = output.split('\n')
    def artifacts = []
    //can not use foreach loop, because it does not work with Pipeline CPS Transform see [JENKINS-27421]
    for (int j = 0; j < lines.length; j++) {
        def line = lines[j]
        if (line.startsWith("[INFO] Installing") && !line.contains(" at end") && !line.contains("Installing artifact ")) {
            def path = line.split(" ")[4]
            def i = path.indexOf(repo)
            artifacts.add(path.substring(i))
        }
    }
    artifacts.join(',')
}

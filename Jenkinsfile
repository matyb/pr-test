import groovy.json.JsonSlurper;
 
node {
  
  stage('print pr target branch name'){
    def jsonString = curl("https://api.github.com/repos/matyb/pr-test/pulls/" + env.BRANCH_NAME.replace("PR-", ""))
    def slurper = new JsonSlurper()
    def json = slurper.parseText(jsonString)
    echo "PULL REQUEST TO: " + json.base.ref
  }
  
}
 
def curl(url) {
  try{
    sh "curl -X GET \"${url}\" -o output.json"
    def workspace = pwd()
    return new File("$workspace/output.json").text
  } finally {
    sh "rm output.json"
  }
}


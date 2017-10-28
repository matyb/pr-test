#! /usr/bin/groovy

import java.text.SimpleDateFormat
import groovy.json.JsonSlurper


node {

  sh "set +x"

  def branches = []
  stage('get "release" branches and sort them chronologically'){
    def jsonString = httpRequest('https://api.github.com/repos/matyb/pr-test/contents/ci').content
    def json = new JsonSlurper().parseText(jsonString)

    branches = json.collect { it.name.replace(".properties", "") }
    sortBranches(branches)
  }

  stage('clone repo'){
    sh "rm -rf pr-test"
    sh "git clone https://github.com/matyb/pr-test.git"
  }

  stage('configure repo'){
    dir('pr-test') {
      sh "git config user.email mathewkbentley@gmail.com"
      sh "git config user.name matyb"
      withCredentials([usernameColonPassword(credentialsId: '00afad12-5723-4868-b1c2-ff78c092345a', variable: 'USERPASS')]){
        sh "git remote add with-user-pass https://$USERPASS@github.com/matyb/pr-test.git"
      }
    }
  }

  dir('pr-test') {
    for(int i = 1;  i < branches.size(); i++){
      def head = branches.get(i - 1)
      def base = branches.get(i)
      def subBranch = "merging-$head-to-$base"
      stage(subBranch.replace("-", " ")) {
        withCredentials([usernameColonPassword(credentialsId: '00afad12-5723-4868-b1c2-ff78c092345a', variable: 'USERPASS')]){
            echo "\n\n======================================================"
            echo "merging $head -> $base; $subBranch"
            echo "======================================================\n\n"

            deleteBranch(subBranch)
            copyHead(head, subBranch)



            def conflicts = []
            try {
              def mergingOutput = sh(returnStdout: true, script: "git checkout $base && git merge origin/$head --no-ff --no-commit")
              if(!mergingOutput.contains("Already up-to-date.")){
                echo "****** Changes to merge found. No conflicts. ******"
              } else {
                echo "****** Branch is already up to date... nothing to do. ******"
              }
            } catch (x) {
              conflicts = sh(returnStdout: true, script: "git status").split("\n").findAll{it.startsWith("\tboth ")}.collect{it.split(":      ").last().trim()}
              echo "****** Found conflicts: " + conflicts.toString() + " ******"
              def windows = []
              for(int j = 0; j < conflicts.size(); j++){
                def blame = sh(returnStdout: true, script: "git blame ${conflicts[j]}").split("\n")
                def start = null
                def middle = null
                for(int k = 0; k < blame.size(); k++){
                    def line = blame[k]
                    if(line.contains(' <<<<<<< ')){
                      start = k
                    }
                    if(line.contains(') =======')){
                      middle = k
                    }
                    if(line.contains(' >>>>>>> ')){
                      windows += ['start': start, 'middle': middle, 'end': k]
                    }
                }
                echo "BLAME: ${blame.toString()}"
              }
              echo "WINDOWS: ${windows.toString()}"
              def shas = []
            } finally {
              sh "git reset --hard origin/$subBranch"
            }
        }
      }
    }
  }
}

@NonCPS
def sortBranches(branches){
  branches.sort {
    def index = ['master', 'develop'].indexOf(it)
    return index > -1 ? Integer.MIN_VALUE + index : new SimpleDateFormat("MMMyyyy").parse(it).time
  }
}

def deleteBranch(branch){
    try{ //delete local branch
      sh "git branch -D $branch"
    } catch (x){} // who cares! didn't want the branch anyway

    try{ //delete remote branch
      sh "git push with-user-pass --delete $branch"
    } catch (x){} // who cares! didn't want the branch anyway
}

def copyHead(head, copy) {
    sh "git checkout -b $copy origin/$head"
    sh "git push -u with-user-pass $copy"
}

def filesChanged(subBranch, base){
  return sh(returnStdout: true, script: "git checkout $subBranch && git cherry $base")
}

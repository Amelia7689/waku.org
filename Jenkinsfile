pipeline {
  agent { label 'linux' }

  options {
    disableConcurrentBuilds()
    /* manage how many builds we keep */
    buildDiscarder(logRotator(
      numToKeepStr: '20',
      daysToKeepStr: '30',
    ))
  }

  environment {
    GIT_COMMITTER_NAME = 'status-im-auto'
    GIT_COMMITTER_EMAIL = 'auto@status.im'
    PROD_SITE = 'waku.org'
    DEV_SITE  = 'dev.waku.org'
    DEV_HOST  = 'jenkins@node-01.do-ams3.sites.misc.statusim.net'
    SCP_OPTS  = 'StrictHostKeyChecking=no'
  }

  stages {
    stage('Install') {
      steps {
        sh "yarn install"
      }
    }

    stage('Build') {
      steps {
        /* Issues from waku-org/bounties are fetched. */
        withCredentials([
          string(
            credentialsId: 'waku-org-bounties-access-gh-token',
            variable: 'GITHUB_ACCESS_TOKEN'
          ),
        ]) {
          sh 'yarn build'
        }
        sh "echo ${env.PROD_SITE} > build/CNAME"
      }
    }

    stage('Publish Prod') {
      when { expression { env.GIT_BRANCH ==~ /.*master/ } }
      steps {
        sshagent(credentials: ['status-im-auto-ssh']) {
          sh "ghp-import -p build"
        }
      }
    }

    stage('Publish Devel') {
      when { expression { env.GIT_BRANCH ==~ /.*develop/ } }
      steps {
        sshagent(credentials: ['jenkins-ssh']) {
          sh """
            rsync -e 'ssh -o ${SCP_OPTS}' -r --delete build/. \
              ${env.DEV_HOST}:/var/www/${env.DEV_SITE}/
          """
        }
      }
    }
  }

  post {
    cleanup { cleanWs() }
  }
}

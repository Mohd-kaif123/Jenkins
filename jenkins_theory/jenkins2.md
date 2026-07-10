# ========================================================================================================================================================

# 4. Important Additional Blocks (Production-level)
pipeline {
    agent any

    environment {
        NODE_ENV = 'production'
        API_KEY = credentials('my-api-key-id')
    }

    options {
        timeout(time: 30, unit: 'MINUTES')
        timestamps()
    }

    triggers {
        pollSCM('H/5 * * * *')
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/user/repo.git'
            }
        }
        stage('Build') {
            steps {
                sh 'npm install'
            }
        }
        stage('Test') {
            steps {
                sh 'npm test'
            }
        }
        stage('Deploy') {
            when {
                branch 'main'
            }
            steps {
                sh './deploy.sh'
            }
        }
    }

    post {
        always {
            echo 'Pipeline finished.'
        }
        success {
            echo 'Deployment successful!'
        }
        failure {
            mail to: 'team@example.com', subject: 'Build Failed', body: 'Check Jenkins logs.'
        }
    }
}

# In naye blocks ka breakdown:

1) environment { }

- Pipeline-wide environment variables define karta hai — saare stages access kar sakte hain
- credentials('my-api-key-id') — Jenkins ke Credentials Manager se securely secret fetch karta hai (jaise GitHub Actions me secrets.MY_KEY)
- Why important: Hardcode karna secrets ko file me — bahut bada security risk. Isliye Jenkins credentials store use karte hain.

2) options { }

- Pipeline-level behavior settings
- timeout(30 min) — agar pipeline itne time me complete na ho to force-stop, resources waste hone se bachata hai
- timestamps() — console logs me time add karta hai, debugging me kaam aata hai

3) triggers { }

- Batata hai pipeline automatically kab trigger ho
- pollSCM('H/5 * * * *') — cron syntax hai, har 5 min me Git repo check karega changes ke liye
- Real world me usually githubPush() trigger use hota hai — jab bhi push ho, turant chale (webhook based, polling se better)

4) when { branch 'main' }

- Conditional execution — is stage ko sirf tab chalao jab condition true ho
- Yahan sirf main branch pe push hone par Deploy stage chalega, baaki branches pe skip
- Real world use case: Feature branches pe sirf build+test chale, deploy sirf production branch pe ho

5) post { }

- Pipeline ke end me kya karna hai — chahe pass ho ya fail
- always — hamesha chalega
- success — sirf jab pipeline pass ho
- failure — sirf jab pipeline fail ho, yahan email notification bhej rahe hain
- Real world: Slack/email notifications, cleanup tasks, artifact archiving

# ========================================================================================================================================================


# 5. Execution Flow (Top to Bottom)

- Jenkins job trigger hota hai (push, poll, ya manual)
- Jenkins Jenkinsfile ko repo se checkout karta hai (agar SCM-based pipeline hai)
- agent block decide karta hai kaunse machine pe chalega
- environment variables set hote hain
- Har stage sequentially chalta hai — agar ek fail hui to (by default) baaki stages skip ho jaati hain
- Har stage ke andar steps top-to-bottom execute hote hain
- Pipeline complete hone ke baad post block chalta hai (result ke hisaab se)
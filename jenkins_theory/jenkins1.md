# 1. Jenkinsfile kya hai aur iska purpose
Jenkinsfile ek text file hai (no extension, bas naam Jenkinsfile) jo Jenkins ko batati hai ki tumhara CI/CD pipeline kaise chalega — build, test, deploy sab steps.
Purpose:

Pipeline ko "as code" define karna — matlab pipeline bhi version control (Git) me rehti hai
Manual Jenkins UI configuration (jise "Freestyle Job" bolte hain) ki jagah leta hai
Reproducible, reviewable, aur trackable pipeline banata hai — jaise tum code review karte ho, waise hi pipeline changes bhi review ho sakte hain

GitHub Actions me tumne .github/workflows/*.yml dekha hai — Jenkinsfile uska Jenkins equivalent hai, bas syntax Groovy hai YAML nahi.

# ========================================================================================================================================================

# 2. Do Types ke Syntax
Jenkins me do tarah ki pipeline likh sakte ho:-

| Feature | Declarative Pipeline | Scripted Pipeline |
| :--- | :--- | :--- |
| **Kaise likhte hain** | Structured, fixed keywords use hote hain (`pipeline`, `stages`, `steps`). | Pure Groovy code hota hai, aur ye `node {}` block se start hota hai. |
| **Kab use karte hain** | 90% real projects me yahi use hota hai kyunki ye simple aur readable hai. | Jab complex logic aur custom control flow ki zaroorat ho, tab use karte hain. |

Interview me almost hamesha Declarative poochte hain kyunki industry me yahi standard hai. Hum isi pe focus karenge.

# ========================================================================================================================================================

# 3. Basic Structure — Line by Line
pipeline {
    agent any

    stages {
        stage('Build') {
            steps {
                echo 'Building the application...'
                sh 'npm install'
            }
        }
        stage('Test') {
            steps {
                echo 'Running tests...'
                sh 'npm test'
            }
        }
        stage('Deploy') {
            steps {
                echo 'Deploying application...'
                sh './deploy.sh'
            }
        }
    }
}

# Line-by-line breakdown:
1) pipeline { }

- Ye root block hai — har declarative Jenkinsfile isi se start hoti hai
- Iske andar sab kuch (agent, stages, post, environment) define hota hai
- Effect of removing: Agar ye missing ho, Jenkins error dega — "declarative pipeline must have pipeline block". Ye mandatory hai, GitHub Actions ke jobs: jaisa top-level required key.


2) agent any

- Batata hai pipeline kaha execute hogi — kaunse machine/node pe
- any matlab Jenkins available kisi bhi agent (worker machine) pe chala dega
- Options: agent none (per-stage agent define karoge), agent { label 'linux' } (specific machine), agent { docker { image 'node:18' } } (container ke andar run)
- Why important: Bina agent ke Jenkins ko pata nahi chalega ki commands kaha execute karne hain. Ye tumhare GitHub Actions ke runs-on: ubuntu-latest jaisa concept hai.

3) stages { }

- Container block jisme saare stages (phases) define hote hain
Ek pipeline me kam se kam ek stage zaroor hona chahiye, warna Jenkins fail hoga

4) stage('Build') { }

- Ek logical phase/step group hai pipeline ka
- 'Build' sirf ek label/name hai — Jenkins UI me yahi naam dikhega har stage box ke upar
- Har stage independent visually track hota hai (green tick / red cross Jenkins dashboard pe)
- Real world: Alag alag stages banate hain taaki pipeline ka flow clearly dikhe — kaunsa step pass hua, kaunsa fail. Debugging bahut aasan ho jaati hai.

5) steps { }

- Har stage ke andar steps block hota hai — yahi actual commands likhte hain
- Iske bina stage ka koi matlab nahi — steps hi wo cheez hai jo real kaam karti hai

6) echo 'Building...'

- Simple print statement — console log jaisa, sirf information/debugging ke liye
- Isse pipeline delete karo to koi functional effect nahi, sirf logs me clarity kam hogi

7) sh 'npm install'

- sh step Linux/Mac shell command execute karta hai (Windows pe bat use hota hai)
- Ye string ke andar koi bhi normal bash command daal sakte ho
- Effect of breaking: Agar command fail hoti hai (non-zero exit code), Jenkins automatically pura stage aur pipeline FAILED mark kar dega — bilkul GitHub Actions ke steps jaisa behavior


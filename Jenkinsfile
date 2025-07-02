pipeline {
    agent any

    parameters {
        choice(
            name: 'BRANCH_NAME',
            choices: ['dev', 'main', 'test'],
            description: 'Select the Git branch to build'
        )
    }

    environment {
        PROJECT_NAME = "Match_Matrix"
        SCHEME_NAME = "Match Matrix"
        PROJECT_FILE = "Match_Matrix.xcodeproj"
        EXPORT_OPTIONS_PLIST = "ExportOptions.plist"
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: "${params.BRANCH_NAME}",
                    credentialsId: 'git-clima-credential',
                    url: 'https://github.com/morahakim/matrix-mania.git'
            }
        }

        stage('Install Dependencies') {
            steps {
                sh '''
                set -e
                if [ -f "Podfile" ]; then
                    pod install
                fi
                '''
            }
        }

        stage('Build & Export IPA') {
            steps {
                sh '''
                set -e

                xcodebuild clean -project "$PROJECT_FILE" -scheme "$SCHEME_NAME" -configuration Release

                xcodebuild archive \
                    -project "$PROJECT_FILE" \
                    -scheme "$SCHEME_NAME" \
                    -archivePath "build/Match Matrix.xcarchive" \
                    -destination 'generic/platform=iOS' \
                    -allowProvisioningUpdates

                xcodebuild -exportArchive \
                    -archivePath "build/Match Matrix.xcarchive" \
                    -exportPath "build/IPA" \
                    -exportOptionsPlist "$EXPORT_OPTIONS_PLIST" \
                    -allowProvisioningUpdates

                echo "IPA directory content:"
                ls -la build/IPA
                '''
            }
        }

        stage('Archive IPA') {
            steps {
                archiveArtifacts artifacts: 'build/IPA/*.ipa', fingerprint: true
            }
        }

        stage('Upload to TestFlight') {
    steps {
        withCredentials([
            file(credentialsId: 'APPSTORE_API_KEY', variable: 'API_KEY_FILE'),
            string(credentialsId: 'APP_STORE_API_KEY', variable: 'APP_STORE_KEY_ID'),
            string(credentialsId: 'APP_STORE_API_ISSUER_ID', variable: 'APP_STORE_ISSUER_ID')
        ]) {
            sh '''
            export LANG=en_US.UTF-8
            export LC_ALL=en_US.UTF-8
            export PATH="$HOME/.fastlane/bin:/opt/homebrew/bin:$PATH"

            fastlane beta \
            key_id:$APP_STORE_KEY_ID \
            issuer_id:$APP_STORE_ISSUER_ID \
            key_filepath:$API_KEY_FILE
            '''
        }
    }
}

    }

    post {
        always {
            cleanWs()
        }
    }
}

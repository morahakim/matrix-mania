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
        IPA_OUTPUT_PATH = "build/IPA/Match_Matrix.ipa"
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
                    -destination 'generic/platform=iOS'

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
                timeout(time: 20, unit: 'MINUTES') {
                    withCredentials([
                        file(credentialsId: 'APPSTORE_API_KEY', variable: 'API_KEY_FILE'),
                        string(credentialsId: 'APP_STORE_API_KEY', variable: 'APP_STORE_KEY_ID'),
                        string(credentialsId: 'APP_STORE_API_ISSUER_ID', variable: 'APP_STORE_ISSUER_ID')
                    ]) {
                        sh """
                        export PATH="\$HOME/.fastlane/bin:/opt/homebrew/bin:/usr/local/bin:/usr/bin:/bin:/usr/sbin:/sbin"
                        export LANG=en_US.UTF-8
                        export LC_ALL=en_US.UTF-8

                        export IPA_OUTPUT_PATH="$IPA_OUTPUT_PATH"
                        export APP_STORE_KEY_ID="$APP_STORE_KEY_ID"
                        export APP_STORE_ISSUER_ID="$APP_STORE_ISSUER_ID"
                        export API_KEY_FILE="$API_KEY_FILE"

                        echo "🚀 Running fastlane..."
                        fastlane beta
                        """
                    }
                }
            }
        }
    }
}

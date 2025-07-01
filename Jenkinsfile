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
                    url: 'git@github.com:morahakim/matrix-mania.git'
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
                    -archivePath build/$PROJECT_NAME.xcarchive \
                    -destination 'generic/platform=iOS' \
                    -allowProvisioningUpdates

                xcodebuild -exportArchive \
                    -archivePath build/$PROJECT_NAME.xcarchive \
                    -exportPath build/IPA \
                    -exportOptionsPlist "$EXPORT_OPTIONS_PLIST" \
                    -allowProvisioningUpdates
                '''
            }
        }

        stage('Archive IPA') {
            steps {
                archiveArtifacts artifacts: 'build/IPA/*.ipa', fingerprint: true
            }
        }
    }
}

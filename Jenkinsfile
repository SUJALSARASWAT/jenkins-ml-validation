pipeline {
    agent any

    environment {
        IMAGE_NAME = "sujal0307/wine-quality-predictor:latest"
        CONTAINER_NAME = "wine_test_container"
        PORT = "8000"
    }

    stages {

        stage('Pull Image') {
            steps {
                sh "docker pull $IMAGE_NAME"
            }
        }

        stage('Run Container') {
            steps {
                sh "docker run -d -p $PORT:8000 --name $CONTAINER_NAME $IMAGE_NAME"
            }
        }

        stage('Wait for Service Readiness') {
            steps {
                script {
                    timeout(time: 60, unit: 'SECONDS') {
                        waitUntil {
                            def result = sh(
                                script: "curl -s http://localhost:$PORT/docs || true",
                                returnStdout: true
                            )
                            return result.contains("Swagger")
                        }
                    }
                }
            }
        }

        stage('Valid Inference Request') {
            steps {
                script {
                    def response = sh(
                        script: """
                        curl -s -X POST http://localhost:$PORT/predict \
                        -H "Content-Type: application/json" \
                        -d '{"features":["7.4","0.7","0","1.9","0.076"]}'
                        """,
                        returnStdout: true
                    )

                    echo "Valid Response: ${response}"

                    if (!response.trim()) {
                        error("No prediction returned")
                    }
                }
            }
        }

        stage('Invalid Request') {
            steps {
                script {
                    def response = sh(
                        script: """
                        curl -s -X POST http://localhost:$PORT/predict \
                        -H "Content-Type: application/json" \
                        -d '{"wrong_key":[1,2,3]}'
                        """,
                        returnStdout: true
                    )

                    echo "Invalid Response: ${response}"

                    if (!response.toLowerCase().contains("error") &&
                        !response.toLowerCase().contains("detail")) {
                        error("API did not return error for invalid input")
                    }
                }
            }
        }

        stage('Stop Container') {
            steps {
                sh """
                docker stop $CONTAINER_NAME || true
                docker rm $CONTAINER_NAME || true
                """
            }
        }
    }

    post {
        success {
            echo "Pipeline PASSED ✅"
        }
        failure {
            echo "Pipeline FAILED ❌"
        }
    }
}

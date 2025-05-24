pipeline {
    agent any
    triggers {
        githubPush()
    }

    environment {
        GITHUB_REPO_URL = 'https://github.com/mohitgupta02/Healthcare.git'
        DOCTOR_DOCKER_IMAGE = 'mohitgupta02/doctor-service'
        PATIENT_DOCKER_IMAGE = 'mohitgupta02/patient-service'
        APPOINTMENT_DOCKER_IMAGE = 'mohitgupta02/appointment-service'
        NOTIFICATION_DOCKER_IMAGE = 'mohitgupta02/notification-service'
    }

    stages {
        stage('Checkout') {
            steps {
                script {
                    git branch: 'main', url: "${GITHUB_REPO_URL}"
                }
            }
        }

        stage('Build Maven JARs') {
            steps {
                script {
                    dir('DoctorService') {
                        sh 'mvn clean package -DskipTests'
                    }
                    dir('PatientService') {
                        sh 'mvn clean package -DskipTests'
                    }
                    dir('AppointmentService') {
                        sh 'mvn clean package -DskipTests'
                    }
                    dir('NotificationService') {
                        sh 'mvn clean package -DskipTests'
                    }
                }
            }
        }

        stage('Build Docker Images') {
            steps {
                script {
                    def doctorImage = docker.build("${DOCTOR_DOCKER_IMAGE}", './DoctorService/')
                    def patientImage = docker.build("${PATIENT_DOCKER_IMAGE}", './PatientService/')
                    def appointmentImage = docker.build("${APPOINTMENT_DOCKER_IMAGE}", './AppointmentService/')
                    def notificationImage = docker.build("${NOTIFICATION_DOCKER_IMAGE}", './NotificationService/')
                }
            }
        }

        stage('Push to DockerHub') {
            steps {
                script {
                    docker.withRegistry('', '9ff341d3-3602-49bd-89e3-9a3f0eb6d030') {
                        docker.image("${DOCTOR_DOCKER_IMAGE}").push()
                        docker.image("${PATIENT_DOCKER_IMAGE}").push()
                        docker.image("${APPOINTMENT_DOCKER_IMAGE}").push()
                        docker.image("${NOTIFICATION_DOCKER_IMAGE}").push()
                    }
                }
            }
        }

        stage('Apply Kubernetes Manifests with Ansible') {
            steps {
                script {
                    withEnv(["ANSIBLE_HOST_KEY_CHECKING=False"]) {
                        ansiblePlaybook(
                            playbook: 'ansible-playbook.yaml',
                            inventory: 'inventory',
                            vaultCredentialsId: 'ansible_vault_pass'
                        )
                    }
                }
            }
        }
    }
}

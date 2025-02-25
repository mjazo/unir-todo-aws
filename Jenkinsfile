pipeline {
    agent any

    stages {
        stage('GetCode') {
            steps {
                cleanWs()
                git 'https://github.com/mjazo/unir-todo-aws.git'
                sh '''
                    ls -la

                    wget -O samconfig.toml https://raw.githubusercontent.com/mjazo/unir-todo-list-aws-config/production/samconfig.toml

                    cat samconfig.toml
                '''
            }
        }
        stage('StaticTest') {
            steps {
                sh '''
                    python3 -m flake8 --format=pylint --exit-zero src > flake8.out
                    python3 -m bandit --exit-zero -r ./src -f custom -o bandit.out --msg-template "{abspath}:{line}: {severity}: {test_id}: {msg}"
                '''
                recordIssues tools: [flake8(name: 'Flake8', pattern: 'flake8.out'),pyLint(name: 'Bandit', pattern: 'bandit.out')]
            }
        }
        stage('Deploy') {
            steps {
                sh '''
                    export PATH="/usr/bin:$PATH"
                    export PYTHONPATH="/usr/bin/python3.10"
                    sam build
                    sam validate --region=us-east-1
                    sam deploy --config-env production --force-upload --no-fail-on-empty-changeset
                '''
            }
        }
        stage('RestTest') {
            steps {
                sh '''
                    BASE_URL=$(aws cloudformation describe-stacks --stack-name todo-list-aws-production --query "Stacks[0].Outputs[0].OutputValue" --output text)

                    export BASE_URL

                    sleep 15

                    python3 -m pytest -s test/integration/todoApiTest.py
                '''
            }
        }
    }
}

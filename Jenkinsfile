// vim: set filetype=groovy:

pipeline {
  agent any
  environment {
    TF_WORKSPACE = 'default' //Sets the Terraform Workspace
    TF_IN_AUTOMATION = 'true'
    TERRAFORM_HOME = tool name: 'terraform-0.14.4', type: 'terraform'
    BUCKET = 'terraform'
    ACTION = params.ACTION
  }
  parameters {
    choice (name: 'ACTION',
	    choices: ['plan', 'apply', 'destroy'],
      description: 'Terraform stage name to execute')
    // string (name: 'ENV_NAME',
    //   defaultValue: 'tf-customer1',
    //   description: 'Env or Customer name')
  }
  stages {
    stage('Terraform Init') {
      steps {
        git branch: 'develop', url: 'https://github.com/dzintars/infra.git'
        dir('./terraform/env/dev') {
          sh '$TERRAFORM_HOME/terraform --version'
          withVault(
            configuration: [
              timeout: 60,
              vaultCredentialId: 'vault-token',
              vaultUrl: 'https://vault.oswee.com'
            ],
            vaultSecrets: [
              [path: 'oswee/minio',
                secretValues: [
                  [envVar: 'MINIO_ACCESS_KEY', vaultKey: 'access_key'],
                  [envVar: 'MINIO_SECRET_KEY', vaultKey: 'secret_key'],
                ],
              ],
            ]
          ) {
            script {
              sh """#!/bin/bash
                ${env.TERRAFORM_HOME}/terraform init -backend-config=access_key=${env.MINIO_ACCESS_KEY} -backend-config=secret_key=${env.MINIO_SECRET_KEY} -backend-config=bucket=${env.BUCKET}
              """
            }
          }
        }
      }
    }
    stage('Terraform Plan') {
      when { anyOf
        {
          environment name: 'ACTION', value: 'plan';
          environment name: 'ACTION', value: 'apply';
        }
      }
      steps {
        dir('./terraform/env/dev') {
          // sh "${env.TERRAFORM_HOME}/terraform plan -out=tfplan -input=false -var-file='terraform.tfvars'"
          withVault(
            configuration: [
              timeout: 60,
              vaultCredentialId: 'vault-token',
              vaultUrl: 'https://vault.oswee.com'
            ],
            vaultSecrets: [
              [path: 'oswee/vault',
                secretValues: [
                  [envVar: 'VAULT_TOKEN', vaultKey: 'token'],
                ],
              ]
            ]
          ) {
            script {
              // sh 'pwd'
              sh """#!/bin/bash
                ${env.TERRAFORM_HOME}/terraform plan
              """
            }
          }
        }
      }
    }
    stage('Terraform Apply') {
      when { anyOf
        {environment name: 'ACTION', value: 'apply';}
      }
      steps {
        dir('./terraform/env/dev') {
          withVault(
            configuration: [
              timeout: 60,
              vaultCredentialId: 'vault-token',
              vaultUrl: 'https://vault.oswee.com'
            ],
            vaultSecrets: [
              [path: 'oswee/vault',
                secretValues: [
                  [envVar: 'VAULT_TOKEN', vaultKey: 'token'],
                ],
              ]
            ]
          ) {
            script {
              sh """#!/bin/bash
                ${env.TERRAFORM_HOME}/terraform apply -input=false -auto-approve
              """
            }
          }
        }
        // input 'Apply Plan'
      }
    }
    stage('Terraform Destroy') {
      when { anyOf
        {environment name: 'ACTION', value: 'destroy';}
      }
      steps {
        dir('./terraform/env/dev') {
          withVault(
            configuration: [
              timeout: 60,
              vaultCredentialId: 'vault-token',
              vaultUrl: 'https://vault.oswee.com'
            ],
            vaultSecrets: [
              [path: 'oswee/vault',
                secretValues: [
                  [envVar: 'VAULT_TOKEN', vaultKey: 'token'],
                ],
              ]
            ]
          ) {
            script {
              sh """#!/bin/bash
                ${env.TERRAFORM_HOME}/terraform destroy -input=false -auto-approve
              """
            }
          }
        }
      }
    }
    stage('Bazel build') {
      steps {
        sh 'bazelisk --version'
      }
    }
  }
  // post {
  //   always {
  //     cleanWs()
  //   }
  // }
}

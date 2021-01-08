pipeline {
  agent any
  environment {
    TF_WORKSPACE = 'default' //Sets the Terraform Workspace
    TF_IN_AUTOMATION = 'true'
    TERRAFORM_HOME = tool name: 'terraform-0.14.4', type: 'terraform'
    BUCKET = 'terraform'
  }
  stages {
    stage('1 Install dmacvicar/libvirt plugin') {
      steps {
        dir('~/.local/share/terraform/plugins/registry.terraform.io/dmacvicar/libvirt/0.6.3/linux_amd64') {
          git 'https://github.com/dmacvicar/terraform-provider-libvirt.git'
          sh 'su make'
          sh 'ls -lah'
        }
      }
    }
    stage('2 Terraform Init') {
      steps {
        dir('./terraform/env/dev') {
          sh '$TERRAFORM_HOME/terraform --version'
          withVault(configuration: [timeout: 60, vaultCredentialId: 'vault-root-token', vaultUrl: 'https://vault.oswee.com'], vaultSecrets: [[path: 'oswee/minio', secretValues: [[envVar: 'MINIO_ACCESS_KEY', vaultKey: 'access_key'], [envVar: 'MINIO_SECRET_KEY', vaultKey: 'secret_key']]]]) {
            script {
              sh """#!/bin/bash
                ${env.TERRAFORM_HOME}/terraform init -backend-config=access_key=${env.MINIO_ACCESS_KEY} -backend-config=secret_key=${env.MINIO_SECRET_KEY} -backend-config=bucket=${env.BUCKET}
              """
            }
          }
        }
      }
    }
    // stage('2 Terraform Plan') {
    //   steps {
    //     sh "cd ./terraform/env/dev && ${env.TERRAFORM_HOME}/terraform plan -out=tfplan -input=false -var-file='terraform/env/dev/terraform.tfvars'"
    //   }
    // }
    // stage('3 Terraform Apply') {
    //   steps {
    //     input 'Apply Plan'
    //     sh "${env.TERRAFORM_HOME}/terraform apply -input=false tfplan"
    //   }
    // }
  }
}

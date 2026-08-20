# -

name: "Terraform CI/CD"

on:
  push:
    branches:
      - main
    paths:
      - "env/ongcp/**"
      - ".github/workflows/terraform-ongcp.yml"

  pull_request:
    branches:
      - main
    paths:
      - "env/ongcp/**"
      - ".github/workflows/terraform-ongcp.yml"

  workflow_dispatch:

jobs:
  terraform:
    name: "Terraform Deploy"
    runs-on: self-hosted

    defaults:
      run:
        working-directory: ./env/ongcp

    steps:
      # 1. GitHub Repo 取得
      - name: Checkout
        uses: actions/checkout@v4

      # 2. Terraform セットアップ
      - name: Setup Terraform
        uses: hashicorp/setup-terraform@v3
        with:
          terraform_version: 1.5.4

      # 3. Terraform Init
      - name: Terraform Init
        run: terraform init

      # 4. Terraform Validate
      - name: Terraform Validate
        run: terraform validate

      # 5. Terraform Plan
      - name: Terraform Plan
        env:
          TF_VAR_db_password: ${{ secrets.DB_PASSWORD }}
        run: terraform plan -no-color

      # 6. Terraform Apply
      # main への push または手動実行時のみリソースを作成
      - name: Terraform Apply
        if: github.event_name == 'push' || github.event_name == 'workflow_dispatch'
        env:
          TF_VAR_db_password: ${{ secrets.DB_PASSWORD }}
        run: terraform apply -auto-approve

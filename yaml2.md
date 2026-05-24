name: Build Pipeline

on:
  push:
    branches:
      - develop

jobs:
  build:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout repository
        uses: actions/checkout@v3

      - name: Clone project_repo
        run: git clone -b develop project_repo

      - name: Build-Code
        run: echo "Build Successful"
name: Inventory Management Pipeline

on:
  push:
    branches:
      - main

jobs:

  build:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout Source Code
        uses: actions/checkout@v4

      - name: Set Up Java
        uses: actions/setup-java@v4
        with:
          distribution: 'temurin'
          java-version: '17'

      - name: Clean Project
        run: mvn clean

      - name: Compile Source
        run: mvn compile

      - name: Run Unit Tests
        run: mvn test

      - name: Package Application
        run: mvn package

      - name: Upload Build Artifact
        uses: actions/upload-artifact@v4
        with:
          name: inventory-system
          path: target/*.jar

  deploy:
    needs: build
    runs-on: ubuntu-latest

    steps:
      - name: Deploy to Production Server
        run: |
          scp target/inventory-system.jar user@server:/apps/

      - name: Restart Application Service
        run: |
          ssh user@server "systemctl restart inventory-app"
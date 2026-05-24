<?xml version="1.0" encoding="UTF-8"?>

<pipeline name="SampleJavaProjectPipeline">

    <project>
        <name>Inventory Management System</name>
        <repository>
            <url>https://github.com/example/inventory-system.git</url>
            <branch>main</branch>
        </repository>
    </project>

    <stages>

        <!-- Build Stage -->
        <stage name="Build">
            <steps>
                <step>
                    <name>Clean Project</name>
                    <command>mvn clean</command>
                </step>

                <step>
                    <name>Compile Source</name>
                    <command>mvn compile</command>
                </step>
            </steps>
        </stage>

        <!-- Test Stage -->
        <stage name="Test">
            <steps>
                <step>
                    <name>Run Unit Tests</name>
                    <command>mvn test</command>
                </step>
            </steps>
        </stage>

        <!-- Package Stage -->
        <stage name="Package">
            <steps>
                <step>
                    <name>Create JAR File</name>
                    <command>mvn package</command>
                </step>
            </steps>
        </stage>

        <!-- Deployment Stage -->
        <stage name="Deploy">
            <environment>production</environment>

            <steps>
                <step>
                    <name>Upload Artifact</name>
                    <command>
                        scp target/inventory-system.jar user@server:/apps/
                    </command>
                </step>

                <step>
                    <name>Restart Service</name>
                    <command>
                        ssh user@server "systemctl restart inventory-app"
                    </command>
                </step>
            </steps>
        </stage>

    </stages>

    <notifications>
        <email>dev-team@example.com</email>
        <onSuccess>true</onSuccess>
        <onFailure>true</onFailure>
    </notifications>

</pipeline>
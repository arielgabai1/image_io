# ImageIO (Twelvemonkeys)

A robust Java image I/O library and plugin collection originally based on the **Twelvemonkeys** project by Harald Kuhr. This project is structured as a multi-module Maven repository containing common utilities and advanced ImageIO extensions.

## Features

- **Multi-Module Architecture**: Segregates common code (`common`) and specialized Image I/O code (`imageio`).
- **Advanced Image Support**: Extends standard Java Image I/O capabilities for a broader range of complex image formats.
- **Automated CI/CD**: Fully equipped with a declarative Jenkins pipeline (`Jenkinsfile`) to automate building, testing, versioning, and deployment.
- **Automated Releases**: The Jenkins pipeline handles Maven versioning and Git tagging for automated releases to Artifactory.

## Technology Stack

- **Language:** Java
- **Build Tool:** Maven
- **CI/CD:** Jenkins
- **Artifact Repository:** Artifactory

## Getting Started

### Prerequisites

- Java JDK 7+ (Jenkins pipeline utilizes JDK 8)
- Maven (3.6+)

### Local Maven Build

To build the project locally, compile the modules, and run the tests:

```bash
mvn clean package
```

To install the artifacts into your local Maven repository:

```bash
mvn clean install
```

## CI/CD Pipeline Configuration

The project includes a comprehensive `Jenkinsfile` designed for automated continuous integration and deployment. 

It handles two primary workflows:
1. **Snapshot Builds**: Default behavior when no specific release version is provided. It tests and deploys the snapshot artifact to the configured Artifactory snapshots repository.
2. **Release Builds**: When a `Release_Version` parameter is provided strings during the Jenkins job trigger, it updates the `pom.xml`, builds, deploys to the releases repository, and commits/tags a new release directly back to the GitLab repository using SSH credentials. Next, it automatically iterates the POM to the next `-SNAPSHOT` version to prepare for future development.

*Prepared for portfolio showcase highlighting CI/CD automation and Java dependency management.*

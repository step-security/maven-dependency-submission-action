[![StepSecurity Maintained Action](https://raw.githubusercontent.com/step-security/maintained-actions-assets/main/assets/maintained-action-banner.png)](https://docs.stepsecurity.io/actions/stepsecurity-maintained-actions)

# maven-dependency-submission-action

This is a GitHub Action that will generate a complete dependency graph for a Maven project and submit the graph to the GitHub repository so that the graph is complete and includes all the transitive dependencies.

The action will invoke maven using the `com.github.ferstl:depgraph-maven-plugin:4.0.3` plugin to generate JSON output of the complete dependency graph, which is then processed and submitted using the [Dependency Submission Toolkit](https://github.com/github/dependency-submission-toolkit) to the GitHub repository.


## Usage

This action supports multi-module projects report dependencies as coming from their respective `pom.xml` files.


### Pre-requisites
For this action to work properly, you must have the Maven available on PATH (`mvn`) or using a `mvnw` Maven wrapper in your maven project directory. Maven will need to be configured to be able to access and pull your dependencies from whatever sources you have defined (i.e. a properly configured `settings.xml` or all details provided in the POM).

Custom maven `settings.xml` can now be specified as an input parameter to the action.

This action writes information in the repository dependency graph, so if you are using the default token, you need to set the `contents: write` permission to the workflow or job. If you are using a personal access token, this token must have the `repo` scope. ([API used by this action](https://docs.github.com/en/rest/dependency-graph/dependency-submission#create-a-snapshot-of-dependencies-for-a-repository))

### Inputs

* `directory` - The directory that contains the `pom.xml` that will be used to generate the dependency graph from. Defaults to the `github.workspace` which is where the source will check out to by default when using `actions/checkout` .

* `token` - The GitHub token that will be used to submit the generated dependency snapshot to the repository. Defaults to the `github.token` from the actions environment.

* `settings-file` - An optional path to a Maven settings.xml file that you want to use to provide additional configuration to Maven.

* `ignore-maven-wrapper` - An optional `true`/`false` flag parameter to ignore the Maven wrapper (if present) in the maven project directory and instead use the version of Maven from the `PATH`. This is set to `false` by default to use the wrapper if one is present.

* `maven-args` - An optional string value (space separated) options to pass to the maven command line when generating the dependency snapshot. This is empty by default.

* `correlator`: An optional identifier to distinguish between multiple dependency snapshots of the same type. Defaults to the [job_id](https://docs.github.com/en/actions/using-workflows/workflow-syntax-for-github-actions#jobsjob_id) of the current job.

## Examples

Generating and submitting a dependency snapshot using the defaults:

```
- name: Submit Dependency Snapshot
  uses: step-security/maven-dependency-submission-action@v5
```

Upon success it will generate a snapshot captured from Maven POM like;
![Screenshot 2022-08-15 at 09 33 47](https://user-images.githubusercontent.com/681306/184603264-3cd69fda-75ff-4a46-b014-630acab60fab.png)

### Configuring for Matrix-Based Workflows

To ensure that the job parameter of the submission remains unique when the action is being called from a workflow that has a matrix, you can pass a `correlator` to the action. This identifier will be appended to the default correlator propterty of a job, ensuring uniqueness across matrix-based workflows. When dealing with Maven-based Java projects that utilize different `pom.xml` files across matrix jobs, you can specify the `directory` relevant to each matrix job. This ensures that the dependency snapshot accurately reflects the dependencies for each specific configuration.

Example of specifying `pom.xml` files for different matrix jobs:

```yaml
jobs:
  build:
    runs-on: ubuntu-latest
    strategy:
      matrix:
        include:
          - java-version: 8
            directory: project1
          - java-version: 11
            directory: project2
    steps:
    - uses: actions/checkout@v7
    - name: Set up JDK ${{ matrix.java-version }}
      uses: actions/setup-java@v5
      with:
        java-version: ${{ matrix.java-version }}
    - name: Submit Dependency Snapshot
      uses: step-security/maven-dependency-submission-action@v5
       with:
        directory: ${{ matrix.directory }}
        correlator: ${{ github.job }}-${{ matrix.directory }}
```

In this example, the action is configured to use different working directories based on the Java version specified in the matrix. This ensures that the dependency snapshot is accurate for each Java version being tested.

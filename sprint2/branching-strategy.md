## Git Flow

### Branches
- **Main**: contains production ready code
- **Develop**: contains pre-production code with features being developed and tested, new features are based off Develop branch then merged for testing
- **Feature**: when developing a new feature, a new feature branch is created off of the Develop branch, when the feature is completed it is reviewed and merged back into the Develop branch
- **Release**: used for preparing production releases, documentation generation, finishing touches and minor bugs specific to the release are done here
- **Hotfix**: based off the main branch, used for fixing bugs that have made it into the Main branch, important to merge changes made into both the Main and Develop branches to ensure fixes are in future releases

### Process
- create a branch off of Develop for each new feature
- when the feature is completed, it is merged back into develop
- when Develop has enough features for a release, or a release data is approaching, a Release branch is forked off of Develop, no new features are added to the Release after this point, when the Release is ready it is merged into Main as well as Develop to ensure fixes from the Release are included in future development
- if production code in Main requires fixes, a Hotfix branch is forked off of main, the fix is made, and the changes are merged into both Main and Develop

### Pros and Cons
- **Pros**:  
    - various types of branches are intuitive for organization
    - systemic development process allows for efficient testing
    - release branches allow for easy support of multiple versions of production code
- **Cons**:
    - depending on the complexity of the product, the Git flow model may overcomplicate and slow development and release cycle
    - historically not able to support continuous delivery or continuous integration

## Github Flow

### Branches
- **Main**: contains production ready code
- **Feature**: contain work on new features or bug fixes which are reviewed, tested and merged into the Main branch

### Rules
1. anything in the main branch is deployable  
    - the main branch is always stable and safe to deploy from or branch off of
2. create descriptive branches off of main
3. push to named branches constantly
4. open a pull request at any time  
    - pull requests are opened when you think your work is ready to be merged into the Main branch 
    - pull requests can also be used if you are in need of help or advice
5. merge only after pull request review
6. deploy immediately after review

### Pros and Cons
- **Pros**:  
    - simple, allows for continuous delivery and continuous integration
    - good for small teams
- **Cons**:
    - unable to support multiple versions of production code at the same time
    - more susceptible to bugs in production 

## Metamarkets Parent POMs

### How to publish parent POMs to Sonatype

```bash
# Prepare a release for the pom
mvn release:prepare

# Check out the tag for the version we just created
git checkout $(git tag -l --sort=version:refname | tail -1)

# Extract the version for the tag
export version=$(xpath pom.xml "/project/version/text()" 2>/dev/null)

# Create  a symbolic link to the pom with the right version
ln -s pom.xml oss-parent-${version}.xml

# Sign and publish the the pom
mvn gpg:sign-and-deploy-file \
  -Durl=https://oss.sonatype.org/service/local/staging/deploy/maven2/ \
  -DrepositoryId=ossrh \
  -DpomFile=oss-parent-${version}.xml \
  -Dfile=oss-parent-${version}.xml
```

language: r
r:
  - release
dist: trusty  # ubuntu 14.04 has additional libraries available
sudo: required
cache: false
warnings_are_errors: false

# install debian libraries to match R-servers
# update pre-installed packages to latest versions
before_install:
  - sudo apt-get -qq update
  - sudo apt-get install -y libgdal-dev libproj-dev python-protobuf libprotoc-dev libprotobuf-dev libv8-dev librsvg2-dev

r_packages:
  - roxygen2
  - covr

r_github_packages:
  - Displayr/travisCI

script:
  - R CMD build --no-manual --no-build-vignettes --no-resave-data .
  - R CMD check --as-cran --no-manual --no-build-vignettes --no-tests *.tar.gz
  - if [ -d tests/testthat ]; then Rscript --default-packages='datasets,utils,grDevices,graphics,stats,methods' -e 'res<-devtools::test(); df <- as.data.frame(res); pass <- sum(df$failed)==0 && all(!df$error); write.csv(df, file="test_results.csv"); quit(status=1-pass, save="no")'; fi

notifications:
  slack:
    rooms:
      - displayr:FTgSTNHC2rpanhJMGTKMwZXM#github-notifications
    template:
      - "Build <%{build_url}|#%{build_number}> %{result} in %{repository_name}@%{branch} by %{author}: <%{compare_url}|%{commit_message}>"
    on_success: change
    on_failure: always

# Warning notifications and downstream package builds are implemented
# by calling R functions so they can be updated in this package without
# committing a new change to .travis.yml in each repository
after_success: 
  - travis_wait Rscript -e "require(travisCI); NotifyWarnings(); TriggerDownstreamBuilds(); CheckCoverage()"

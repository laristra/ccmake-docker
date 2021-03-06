FROM laurelmcintyre/buildenv:fedora
#see laurelmcintyre/buildenv

#arguments passed from travis
ARG COVERAGE
ARG SONARQUBE
ARG SONARQUBE_TOKEN
ARG SONARQUBE_GITHUB_TOKEN
ARG CC
ARG CXX

#for coverage
ARG CI
ARG TRAVIS
ARG TRAVIS_BRANCH
ARG TRAVIS_JOB_NUMBER
ARG TRAVIS_PULL_REQUEST 
ARG TRAVIS_JOB_ID
ARG TRAVIS_TAG
ARG TRAVIS_REPO_SLUG
ARG TRAVIS_COMMIT
ARG TRAVIS_OS_NAME

RUN rm -rf /home/user/.ccache
COPY git-src /home/user/git-src
COPY ccache/ /home/user/.ccache
COPY sonar/ /home/user/.sonar
USER root
RUN chown -R user:user /home/user/git-src /home/user/.ccache /home/user/.sonar
USER user

#navigate to git-src, then make directory called build
WORKDIR /home/user/git-src
RUN mkdir build

WORKDIR build
RUN ccache -z
RUN cmake ${COVERAGE:+-DENABLE_COVERAGE_BUILD=ON} ..
RUN ${SONARQUBE:+build-wrapper-linux-x86-64 --out-dir bw-output} make VERBOSE=1 -j2
RUN ccache -s
RUN make test
RUN make install DESTDIR=${PWD}
RUN make html
RUN cd .. && if [ ${COVERAGE} ]; then \
   python -m coverxygen --xml-dir build/xml/ --src-dir . --output doxygen.coverage.info; \
   wget -O codecov.sh https://codecov.io/bash; \
   bash codecov.sh -X gcov -f doxygen.coverage.info -F documentation; \
  if [ ${CC} = clang ]; then \
    $HOME/.local/bin/codecov -F ${CC} --gcov-exec "llvm-cov gcov"; \
  else \
    $HOME/.local/bin/codecov -F ${CC}; \
  fi; \
fi && cd -
RUN cd .. && if [ ${SONARQUBE} ]; then \
  [ ${COVERAGE} ] || find . -type f -name '*.gcno' -exec gcov -pb  {} +; \
  sonar-scanner \
    -Dsonar.projectKey=${TRAVIS_REPO_SLUG##*/} \
    -Dsonar.projectName=${TRAVIS_REPO_SLUG#*/} \
    -Dsonar.projectVersion=${TRAVIS_COMMIT} \
    -Dsonar.branch=/${TRAVIS_BRANCH} \
    -Dsonar.links.homepage=http://github.com/${TRAVIS_REPO_SLUG} \
    -Dsonar.links.ci=https://travis-ci.org/${TRAVIS_REPO_SLUG} \
    -Dsonar.links.scm=https://github.com/${TRAVIS_REPO_SLUG} \
    -Dsonar.links.issue=https://github.com/${TRAVIS_REPO_SLUG}/issues \
    -Dsonar.sources=. \
    -Dsonar.exclusions=build/CMakeFiles/**,build/html/**,build/xml/**,tests/** \
    -Dsonar.cfamily.build-wrapper-output=build/bw-output \
    -Dsonar.host.url=https://sonarqube.com \
    -Dsonar.cfamily.gcov.reportsPath=. \
    -Dsonar.organization=${TRAVIS_REPO_SLUG%%/*}-github \
    $([ ${TRAVIS_PULL_REQUEST} != false ] && echo \
       -Dsonar.analysis.mode=preview \
       -Dsonar.github.pullRequest=${TRAVIS_PULL_REQUEST} \
       -Dsonar.github.repository=${TRAVIS_REPO_SLUG} \
       -Dsonar.github.oauth=${SONARQUBE_GITHUB_TOKEN}) \
    -Dsonar.login=${SONARQUBE_TOKEN}; \
fi && cd -
USER root
RUN make install
USER user
WORKDIR /home/user

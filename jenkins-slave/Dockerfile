ARG UBUNTU_FLAVOR=xenial
ARG UBUNTU_DATE=20181113
FROM ubuntu:${UBUNTU_FLAVOR}-${UBUNTU_DATE}
LABEL maintainer="nareshnavinash@gmail.com"

RUN apt -qqy update \
  && apt -qqy install \
    libltdl7 \
    git \
    wget \
    sudo \
    libhavege1 \
    netcat-openbsd \
    pwgen \
    bc \
    unzip \
    bzip2 \
    apt-utils \
    net-tools \
    jq \
    psmisc \
    iproute2 \
    iputils-ping \
    dbus-x11 \
    wget \
    curl \
  && apt -qyy autoremove \
  && rm -rf /var/lib/apt/lists/* \
  && apt -qyy clean

# Setting language
ENV LANG_WHICH en
ENV LANG_WHERE US
ENV ENCODING UTF-8
ENV LANGUAGE ${LANG_WHICH}_${LANG_WHERE}.${ENCODING}
ENV LANG ${LANGUAGE}

RUN apt -qqy update \
  && apt -qqy --no-install-recommends install \
    language-pack-en \
    tzdata \
    locales \
  && locale-gen ${LANGUAGE} \
  && dpkg-reconfigure --frontend noninteractive locales \
  && apt -qyy autoremove \
  && rm -rf /var/lib/apt/lists/* \
  && apt -qyy clean

ARG user=jenkins
ARG group=jenkins
ARG uid=1001
ARG gid=1001

ENV JENKINS_HOME /home/${user}

RUN groupadd -g ${gid} ${group}

# set a user with sudo permission without password
RUN useradd -d "$JENKINS_HOME" -u ${uid} -g ${gid} -m -s /bin/bash ${user} \
         --shell /bin/bash  \
         --create-home \
  && usermod -a -G sudo jenkins \
  && gpasswd -a jenkins video \
  && echo 'jenkins:secret' | chpasswd \
  && useradd extrauser \
         --shell /bin/bash  \
  && usermod -a -G sudo extrauser \
  && gpasswd -a extrauser video \
  && gpasswd -a extrauser jenkins \
  && echo 'extrauser:secret' | chpasswd \
  && echo 'ALL ALL = (ALL) NOPASSWD: ALL' >> /etc/sudoers




USER jenkins
WORKDIR /home/jenkins

# Selenium standalone latest
ARG SEL_DIRECTORY="3.141"
ENV SEL_VER="3.141.59"

RUN echo $SEL_VER
RUN  export SELBASE="https://selenium-release.storage.googleapis.com" \
  && export SELPATH="${SEL_DIRECTORY}/selenium-server-standalone-${SEL_VER}.jar" \
  && wget -nv ${SELBASE}/${SELPATH} \
  && ln -s "selenium-server-standalone-${SEL_VER}.jar" \
           "selenium-server-standalone-3.jar"

LABEL selenium_version "${SEL_VER}"


# Firefox installation
USER root

RUN apt -qqy update \
  && apt -qqy --no-install-recommends install \
    `apt-cache depends firefox | awk '/Depends:/{print$2}'` \
  && rm -rf /var/lib/apt/lists/* \
  && apt -qyy clean


ENV FF_LANG="en-US" \
    FF_BASE_URL="https://archive.mozilla.org/pub" \
    FF_PLATFORM="linux-x86_64" \
    FF_INNER_PATH="firefox/releases"

ARG FF_VER="64.0.2"

ENV FF_COMP="firefox-${FF_VER}.tar.bz2"
ENV FF_URL="${FF_BASE_URL}/${FF_INNER_PATH}/${FF_VER}/${FF_PLATFORM}/${FF_LANG}/${FF_COMP}"
RUN cd /opt \
  && wget -nv "${FF_URL}" -O "firefox.tar.bz2" \
  && bzip2 -d "firefox.tar.bz2" \
  && tar xf "firefox.tar" \
  && rm "firefox.tar" \
  && ln -fs /opt/firefox/firefox /usr/bin/firefox \
  && chown -R jenkins:jenkins /opt/firefox \
  && chmod -R 777 /opt/firefox

LABEL selenium_firefox_version "${FF_VER}"

# GeckoDriver
ARG GECKOD_VER="0.23.0"
ENV GECKOD_URL="https://github.com/mozilla/geckodriver/releases/download"
RUN wget --no-verbose -O geckodriver.tar.gz \
     "${GECKOD_URL}/v${GECKOD_VER}/geckodriver-v${GECKOD_VER}-linux64.tar.gz" \
  && rm -rf /opt/geckodriver* \
  && tar -C /opt -xvzf geckodriver.tar.gz \
  && chmod +x /opt/geckodriver \
  && cp /opt/geckodriver /usr/bin/geckodriver \
  && chown jenkins:jenkins /opt/geckodriver \
  && chown jenkins:jenkins /usr/bin/geckodriver \
  && rm geckodriver.tar.gz

# COPY bin/fail /usr/bin/


# Google Chrome
ARG EXPECTED_CHROME_VERSION="71.0.3578.98"
ENV CHROME_URL="https://dl.google.com/linux/direct" \
    CHROME_BASE_DEB_PATH="/home/jenkins/chrome-deb/google-chrome" \
    GREP_ONLY_NUMS_VER="[0-9.]{2,20}"

LABEL selenium_chrome_version "${EXPECTED_CHROME_VERSION}"

RUN apt -qqy update \
  && mkdir -p chrome-deb \
  && wget -nv "${CHROME_URL}/google-chrome-stable_current_amd64.deb" \
          -O "./chrome-deb/google-chrome-stable_current_amd64.deb" \
  && apt -qyy --no-install-recommends install \
        "${CHROME_BASE_DEB_PATH}-stable_current_amd64.deb" \
  && rm "${CHROME_BASE_DEB_PATH}-stable_current_amd64.deb" \
  && rm -rf ./chrome-deb \
  && apt -qyy autoremove \
  && rm -rf /var/lib/apt/lists/* \
  && apt -qyy clean \
  && export CH_STABLE_VER=$(/usr/bin/google-chrome-stable --version | grep -iEo "${GREP_ONLY_NUMS_VER}") \
  && echo "CH_STABLE_VER:'${CH_STABLE_VER}' vs EXPECTED_CHROME_VERSION:'${EXPECTED_CHROME_VERSION}'" \
  && [ "${CH_STABLE_VER}" = "${EXPECTED_CHROME_VERSION}" ] || fail

# Chrome webdriver
USER jenkins

ARG CHROME_DRIVER_VERSION="2.45"
ENV CHROME_DRIVER_BASE="chromedriver.storage.googleapis.com" \
    CPU_ARCH="64"
ENV CHROME_DRIVER_FILE="chromedriver_linux${CPU_ARCH}.zip"
ENV CHROME_DRIVER_URL="https://${CHROME_DRIVER_BASE}/${CHROME_DRIVER_VERSION}/${CHROME_DRIVER_FILE}"

RUN  wget -nv -O chromedriver_linux${CPU_ARCH}.zip ${CHROME_DRIVER_URL} \
  && unzip chromedriver_linux${CPU_ARCH}.zip \
  && rm chromedriver_linux${CPU_ARCH}.zip \
  && mv chromedriver \
        chromedriver-${CHROME_DRIVER_VERSION} \
  && chmod 755 chromedriver-${CHROME_DRIVER_VERSION} \
  && ln -s chromedriver-${CHROME_DRIVER_VERSION} \
           chromedriver \
  && sudo ln -s /home/jenkins/chromedriver /usr/bin

RUN chown -R ${user}:${user} /home/${user}


# Set Name Servers
COPY /files/resolv.conf /etc/resolv.conf




















# ##################################################################################
#  or
# ##################################################################################

# # chrome
# FROM selenium/node-chrome:3.141.59-europium
# LABEL authors=SeleniumHQ

# USER jenkins

# #====================================
# # Scripts to run Selenium Standalone
# #====================================
# COPY start-selenium-standalone.sh /opt/bin/start-selenium-standalone.sh

# #==============================
# # Supervisor configuration file
# #==============================
# COPY selenium.conf /etc/supervisor/conf.d/


# EXPOSE 4444


# # firefox
# FROM selenium/node-firefox:3.141.59-europium
# LABEL authors=SeleniumHQ

# USER jenkins

# #====================================
# # Scripts to run Selenium Standalone
# #====================================
# COPY start-selenium-standalone.sh /opt/bin/start-selenium-standalone.sh

# #==============================
# # Supervisor configuration file
# #==============================
# COPY selenium.conf /etc/supervisor/conf.d/


# EXPOSE 4444

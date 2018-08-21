FROM node:10.9-stretch

ARG USER_ID=1000

# There's already a user with ID 1000 present that
# we know is safe to run the image as.
RUN if [ "$USER_ID" -ne "1000" ]; then \
      useradd -u $USER_ID -m -U -d /home/app app; \
    fi

USER 1000

WORKDIR /opt/src
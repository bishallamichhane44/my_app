# Base stage
FROM python:3.9-slim AS base

# Install system dependencies
RUN apt-get update && apt-get install -y \
    curl \
    git \
    build-essential \
    libssl-dev \
    libffi-dev \
    python3-dev \
    nodejs \
    npm \
    unzip

# Install Expo CLI
RUN npm install -g expo-cli

# Install Android SDK
ENV ANDROID_HOME /usr/local/android-sdk
RUN mkdir -p $ANDROID_HOME && \
    cd $ANDROID_HOME && \
    curl -o sdk-tools.zip https://dl.google.com/android/repository/commandlinetools-linux-6858069_latest.zip && \
    unzip -q sdk-tools.zip -d cmdline-tools && \
    rm sdk-tools.zip && \
    mv cmdline-tools tools

# Add Android SDK to PATH
ENV PATH $PATH:$ANDROID_HOME/tools/bin

# Install Python dependencies
COPY backend/requirements.txt /app/
RUN pip install --upgrade pip && pip install -r /app/requirements.txt

# Stage for web service
FROM base AS web
COPY backend /app/backend
COPY ./backend/backend/manage.py /app/
WORKDIR /app/backend
EXPOSE 8000
CMD ["python", "manage.py", "runserver", "0.0.0.0:8000"]

# Stage for expo service
FROM base AS expo
COPY frontend /app/frontend
WORKDIR /app/frontend
EXPOSE 19000 19001 19002
CMD ["sh", "-c", "npm install && expo start --tunnel"]

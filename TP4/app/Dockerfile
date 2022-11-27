FROM debian

RUN apt update -y

RUN apt install -y golang git

RUN /usr/bin/git clone https://github.com/Max33270/Golang-HangmanWeb2.git /app

WORKDIR /app

CMD [ "/usr/bin/go", "run", "./main/main.go" ]

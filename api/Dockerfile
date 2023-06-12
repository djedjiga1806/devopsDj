# Build
FROM openjdk:11 as build
WORKDIR /app
COPY Main.java .
RUN javac Main.java

# Run
FROM openjdk:11-jre
WORKDIR /app
COPY --from=build /app/Main.class .
CMD ["java", "Main"]

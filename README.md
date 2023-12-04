\# Docker Kafka Cassandra Setup Guide (with openweathermapAPI)

This guide will walk you through the process of setting up Docker
containers for Kafka and Cassandra, along with producers and consumers,
to create a data pipeline. The data pipeline will fetch weather
information from OpenWeatherMap and store it in Cassandra, with the
option for data visualization.

\## Prerequisites

\- \[Docker\](https://docs.docker.com/get-docker/) - \[WSL (Windows
Subsystem for
Linux)\](https://docs.microsoft.com/en-us/windows/wsl/install) - Access
to \[OpenWeatherMap API\](https://openweathermap.org/api)

\## Setup Steps

1\. \[Download the
repository\](https://rmit.instructure.com/courses/131316/files/34967075?wrap=1)

2\. Open the \`owm-producer/openweathermap_service.cfg\` file and
replace \`your_api_key\` with your OpenWeatherMap API key.

3\. Open a terminal and navigate to your directory:

\`\`\`bash cd your_directory\\docker-kafka-cassandra-main \`\`\`

4\. Create Docker networks for Kafka and Cassandra:

\`\`\`bash docker network create kafka-network docker network create
cassandra-network \`\`\`

5\. Start Cassandra container:

\`\`\`bash docker-compose -f cassandra/docker-compose.yml up -d \`\`\`

6\. Start Kafka container:

\`\`\`bash docker-compose -f kafka/docker-compose.yml up -d \`\`\`

7\. Check if both containers are running:

\`\`\`bash docker ps -a \`\`\`

8\. Access Kafka UI frontend:

Open \[http://localhost:9000\](http://localhost:9000) (username: admin;
password: bigbang)

Add a data pipeline cluster:  - Cluster Name: mycluster  - Cluster
Zookeeper: zookeeper:2181 Enable the following options:  - Enable JMX
Polling  - Poll consumer information  - Enable Active OffsetCache Click
Save.

9\. In WSL, execute:

\`\`\`bash curl -X GET http://localhost:8083/connectors \`\`\`

If the output is \`\[\"twittersink\",\"weathersink\"\]\`, proceed to the
next step. Otherwise, go into the Kafka-connect container shell and run
\`./start-and-wait.sh\`.

10\. Start OpenWeatherMap Producer:

\`\`\`bash docker-compose -f owm-producer/docker-compose.yml up \`\`\`

11\. Start consumers:

\`\`\`bash docker-compose -f consumers/docker-compose.yml up \`\`\`

12\. Check if data is arriving in Cassandra DB:

Open a shell from the Cassandra container:

\`\`\`bash docker exec -it cassandra bash \`\`\`

In the shell, execute Cassandra Query Language Shell:

\`\`\`bash cqlsh \--cqlversion=3.4.4 127.0.0.1 \`\`\`

In Cassandra QL Shell, use the \`kafkapipeline\` database and then
select data from the \`weatherreport\` table:

\`\`\`cql use kafkapipeline; select \* from weatherreport; \`\`\`

13\. Data Visualization for Data in Cassandra:

\`\`\`bash docker-compose -f data-vis/docker-compose.yml up -d \`\`\`

Open \[http://localhost:8888\](http://localhost:8888) for data
visualization.

Now, you have successfully set up the Docker containers for Kafka and
Cassandra, created a data pipeline, and visualized the data. If you
encounter any issues, refer to the troubleshooting section in the
documentation.

\# Faker Data Pipeline Setup Guide (Continue from above)

\## Objective

Set up a Faker Data Pipeline to generate and process synthetic data
using Faker API. The pipeline will produce data using a faker producer,
consume it through Kafka, and store it in Cassandra. Additionally,
you\'ll visualize the data in Jupyter Notebook.

\## Setup Steps

1\. Take down all containers except Cassandra:

\`\`\` docker-compose -f data-vis/docker-compose.yml down \# stop
visualization node docker-compose -f consumers/docker-compose.yml down
\# stop the consumers docker-compose -f owm-producer/docker-compose.yml
down \# stop open weather map producer docker-compose -f
kafka/docker-compose.yml down \# stop zookeeper, broker, kafka-manager
and kafka-connect services \`\`\` 2. Go back to Cassandra Exec by
repeating step 12

In Cassandra QL Shell, use the \`kafkapipeline\` database and create a
table for fakerdata

\`\`\` CREATE TABLE IF NOT EXISTS fakerdata ( id UUID PRIMARY KEY,
address TEXT, city TEXT, color TEXT, company TEXT, country TEXT,
credit_card_number TEXT, date_of_birth DATE, email TEXT, ipv4 INET, job
TEXT, name TEXT, phone_number TEXT, sentence TEXT, text TEXT, word TEXT,
year INT ); \`\`\`

\*\*\_Importance:\_\*\* From this step ahead, let\'s assume that we are
using my docker-kafka-cassandra-main folder, all of the code
modification in this part is specified in the directories below.

\`\*/kafka\` \`\*/faker-producer\` \`\*/consumers\` \`\*/data-vis\`

3\. Start Kafka container:

\`\`\`bash docker-compose -f kafka/docker-compose.yml up -d \`\`\`

4\. Check if both containers are running:

\`\`\`bash docker ps -a \`\`\`

5\. Access Kafka UI frontend:

Open \[http://localhost:9000\](http://localhost:9000) (username: admin;
password: bigbang)

Add a data pipeline cluster:  - Cluster Name: mycluster  - Cluster
Zookeeper: zookeeper:2181 Enable the following options:  - Enable JMX
Polling  - Poll consumer information  - Enable Active OffsetCache Click
Save.

6\. In WSL, execute:

\`\`\`bash curl -X GET http://localhost:8083/connectors \`\`\`

If the output is \`\[\"twittersink\",\"weathersink\"\]\`, proceed to the
next step. Otherwise, go into the Kafka-connect container shell and run
\`./start-and-wait.sh\`.

7\. Start Faker Producer:

\`\`\`bash docker-compose -f faker-producer/docker-compose.yml up \`\`\`

8\. Start consumers:

\`\`\`bash docker-compose -f consumers/docker-compose.yml up \`\`\`

9\. Check if data is arriving in Cassandra DB:

Open a shell from the Cassandra container:

\`\`\`bash docker exec -it cassandra bash \`\`\`

In the shell, execute Cassandra Query Language Shell:

\`\`\`bash cqlsh \--cqlversion=3.4.4 127.0.0.1 \`\`\`

In Cassandra QL Shell, use the \`kafkapipeline\` database and then
select data from the \`weatherreport\` table:

\`\`\`cql use kafkapipeline; select \* from fakerdata; \`\`\`

10\. Data Visualization for Data in Cassandra:

\`\`\`bash docker-compose -f data-vis/docker-compose.yml up -d \`\`\`

Open \[http://localhost:8888\](http://localhost:8888) for data
visualization.

\# Flight Logs Data Pipeline Setup Guide (Continue from above)

\## Objective

Set up a Filght Logs Data Pipeline to generate and process synthetic
data using Mockaroo_Flight Logs API. The pipeline will produce data
using randomly generated data from Mockaroo website achieved by using
schema id and API key from the site, consume it through Kafka, and store
it in Cassandra. Additionally, you\'ll visualize the data in Jupyter
Notebook.

\*\*Data Dictionary\*\*

\- \`flight_id\`: A unique identifier for each flight record. -
\`airline\`: The name of the airline operating the flight. -
\`arrival_airport\`: The code or identifier for the airport of
arrival. - \`arrival_date\`: The date of arrival in the format
\'YYYY-MM-DD\'. - \`arrival_time\`: The time of arrival in the format
\'HH:MM:SS.SSSSSSSSS\'. - \`departure_airport\`: The code or identifier
for the airport of departure. - \`departure_date\`: The date of
departure in the format \'YYYY-MM-DD\'. - \`departure_time\`: The time
of departure in the format \'HH:MM:SS.SSSSSSSSS\'. -
\`flight_duration\`: The duration of the flight in hours. -
\`flight_number\`: The flight number assigned to the flight. -
\`passenger_count\`: The number of passengers on the flight.

\## Setup Steps

1\. Take down all containers except Cassandra:

\`\`\` docker-compose -f data-vis/docker-compose.yml down \# stop
visualization node docker-compose -f consumers/docker-compose.yml down
\# stop the consumers docker-compose -f owm-producer/docker-compose.yml
down \# stop open weather map producer docker-compose -f
kafka/docker-compose.yml down \# stop zookeeper, broker, kafka-manager
and kafka-connect services \`\`\` 2. Go back to Cassandra Exec by
repeating step 12

In Cassandra QL Shell, use the \`kafkapipeline\` database and create a
table for fakerdata

\`\`\` CREATE TABLE IF NOT EXISTS flightlogsdata ( flight_id TEXT
PRIMARY KEY, flight_number INT, departure_airport TEXT, arrival_airport
TEXT, departure_date DATE, arrival_date DATE, departure_time TIME,
arrival_time TIME, airline TEXT, passenger_count INT, flight_duration
DECIMAL ); \`\`\`

\*\*\_Importance:\_\*\* From this step ahead, let\'s assume that we are
using my docker-kafka-cassandra-main folder, all of the code
modification in this part is specified in the directories below.

\`\*/kafka\` \`\*/flightlogs-producer\` \`\*/consumers\` \`\*/data-vis\`

3\. Start Kafka container:

\`\`\`bash docker-compose -f kafka/docker-compose.yml up -d \`\`\`

4\. Check if both containers are running:

\`\`\`bash docker ps -a \`\`\`

5\. Access Kafka UI frontend:

Open \[http://localhost:9000\](http://localhost:9000) (username: admin;
password: bigbang)

Add a data pipeline cluster:  - Cluster Name: mycluster  - Cluster
Zookeeper: zookeeper:2181 Enable the following options:  - Enable JMX
Polling  - Poll consumer information  - Enable Active OffsetCache Click
Save.

6\. In WSL, execute:

\`\`\`bash curl -X GET http://localhost:8083/connectors \`\`\`

If the output is \`\[\"twittersink\",\"weathersink\"\]\`, proceed to the
next step. Otherwise, go into the Kafka-connect container shell and run
\`./start-and-wait.sh\`.

7\. Start Flight Logs Producer:

\`\`\`bash docker-compose -f flightlogs-producer/docker-compose.yml up
-d \`\`\`

8\. Start consumers:

\`\`\`bash docker-compose -f consumers/docker-compose.yml up \`\`\`

9\. Check if data is arriving in Cassandra DB:

Open a shell from the Cassandra container:

\`\`\`bash docker exec -it cassandra bash \`\`\`

In the shell, execute Cassandra Query Language Shell:

\`\`\`bash cqlsh \--cqlversion=3.4.4 127.0.0.1 \`\`\`

In Cassandra QL Shell, use the \`kafkapipeline\` database and then
select data from the \`weatherreport\` table:

\`\`\`cql use kafkapipeline; select \* from flightlogsdata; \`\`\`

10\. Data Visualization for Data in Cassandra:

\`\`\`bash docker-compose -f data-vis/docker-compose.yml up -d \`\`\`

Open \[http://localhost:8888\](http://localhost:8888) for data
visualization.

\# Visualization and Analysis

Descriptions for the charts will be discussed below.

1\. \*\*Top 10 Most Common Job Titles:\*\*  - Description: This bar
chart displays the top 10 most common job titles generated using Faker
library. Each bar represents a job title, and the height of the bar
corresponds to the count of that job title in the dataset.  -
Interpretation: The chart provides an overview of the distribution of
job titles in the generated dataset, highlighting the most frequently
occurring job titles.

2\. \*\*Temperature Trends - Ho Chi Minh City vs. Tokyo:\*\*  -
Description: This line chart compares the average temperatures
(calculated as the mean of max and min temperatures) and \'feels-like\'
temperatures between Ho Chi Minh City and Tokyo over a period of time.
The x-axis represents the date and time, and the y-axis represents the
temperature in Kelvin.  - Interpretation: The chart allows a visual
comparison of temperature trends in Ho Chi Minh City and Tokyo,
providing insights into how the \"feels-like\" temperature compares to
the average temperature over time.

3\. \*\*Top 10 Busiest Routes by Passenger Count:\*\*  - Description:
This bar chart illustrates the top 10 busiest flight routes based on the
total passenger count. Each bar represents a flight route, and the
height of the bar corresponds to the total number of passengers on that
route.  - Interpretation: The chart highlights the routes with the
highest passenger traffic, providing valuable information about the
popularity of different flight connections.

4\. \*\*Top 3 Most Common Airlines:\*\*  - Description: This bar chart
displays the top three most common airlines in the flight logs dataset.
Each bar represents an airline, and the height of the bar corresponds to
the number of flights operated by that airline.  - Interpretation: The
chart offers insights into the dominance of certain airlines in the
dataset, showcasing the airlines that have the highest frequency of
flights.
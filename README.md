# fintech-monitoring

This repository contains configurations and resources for monitoring our fintech project using Grafana and Prometheus. It provides insights into the health and performance of our various services, enabling proactive issue detection and resolution.

## Overview

This monitoring setup is designed to provide a comprehensive view of our microservices architecture, covering key metrics such as:

*   **Resource Utilization:** CPU, memory, and disk usage of each service.
*   **Application Performance:** Request latencies, error rates, and throughput.
*   **Service Health:** Availability, uptime, and potential issues.
*   **Custom Business Metrics:** Metrics relevant to our specific financial operations.

## Technologies

*   **Prometheus:** A time-series database that collects metrics from various sources.
*   **Grafana:** A visualization tool for creating dashboards and alerts based on Prometheus data.
*   **Service Exporters:** Prometheus exporters integrated within our services to expose relevant metrics (e.g., Java Micrometer, Python Prometheus Client).
*   **Docker/Docker Compose:** Used to deploy and orchestrate monitoring components.
*   **(Optional) Alertmanager:** For configuring alerts based on Prometheus rules.

## Getting Started

### Prerequisites

*   Docker and Docker Compose installed.
*   Basic understanding of Prometheus and Grafana.
*   Access to the internal network where the microservices are running.
*  Your microservices should already be configured to expose metrics in the prometheus format through dedicated HTTP endpoints.

### Repository Structure

### Deployment

1.  **Clone the Repository:**
    ```bash
    git clone <repository_url>
    cd fintech-monitoring
    ```

2.  **Configure Prometheus (prometheus/prometheus.yml):**
    *   Review the `scrape_configs` section, and add/modify the targets according to your microservices.
    * The file should already have all the jobs pointing to the target microservices. Ensure that the urls in the configuration correspond to the correct ports exposing metrics of each service.
    *Example:
     ```yaml
     scrape_configs:
        - job_name: "wallet-service"
           scrape_interval: 15s
           static_configs:
             - targets: ["wallet-service:8080"]
    ```

3.  **Configure Grafana (grafana/provisioning/dashboards.yml):**
    * This file enables grafana to load dashboards from the `grafana/dashboards` directory.
    * If you create a new dashboard ensure to update this configuration file.
   
4. **Import dashboards to the grafana/dashboards directory**
     * Dashboards can be created by hand, imported from the internet or even exported from your own Grafana instances.

5.  **Start the Monitoring Stack:**
    ```bash
    docker-compose up -d
    ```

    This command will start Prometheus and Grafana. You can check the status by inspecting the logs with: `docker-compose logs -f`.

6.  **Access Grafana:**
    *   Open your web browser and navigate to `http://localhost:3000`.
    *   The default login credentials are `admin` for both username and password. *Remember to change this in the first login.*

7.  **Explore the Dashboards:**
    *   Go to "Dashboards" in the Grafana UI, and the dashboards will be already loaded through the provisioning mechanism.
    * Start observing the metrics for all your services in the provided dashboards.

## Customization

### Adding New Metrics
1.  **Instrument your services**
    *   Add the needed libraries to expose prometheus metrics through HTTP.
    * For instance, if you have java services, add spring boot actuator + micrometer prometheus to your project and expose it under `/actuator/prometheus`
   
2.  **Add the metrics target to the prometheus.yml configuration**
    *   As explained above, create a new target job pointing to the service you want to monitor.
    
3.  **Create new graphs or dashboards**
    *   Create new dashboards or graphs using grafana, and save the JSON file in `grafana/dashboards`.

### Modifying Alert Rules (Optional)

*   Edit the files under `prometheus/rules` to configure specific alert conditions based on the metrics.
* Check the Prometheus documentation for syntax of the alerting rules.
* If you wish to set up an alertmanager instance to handle the alerts, refer to its official documentation for configuration and deployment steps.

## Contributing

If you want to improve this monitoring setup, please follow these steps:

1.  Fork the repository.
2.  Create a new branch (`git checkout -b feature/your-feature`).
3.  Make your changes and commit them (`git commit -am 'Add some feature'`).
4.  Push to the branch (`git push origin feature/your-feature`).
5.  Create a pull request.

## Contact
Feel free to contact the team responsible for the project if you have any questions or suggestions.

## License

This project is licensed under the [MIT License](LICENSE.txt).
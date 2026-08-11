# GT-CRIVO DefectDojo extensions

Our project has made three main extensions to DefectDojo.  A key requirement our extensions support is that they make *zero* changes to DefectDojo's models or database; this ensures that any deployment using our extensions can be trivially reverted to a vanilla DefectDojo deployment by just changing the codebase.

The extensions are:
1. The ability to group Findings into Problems.
2. The ability for analysts to specify the risk associated with a vulnerability after analyzing it.
3. Integration of complementary metadata to aid analysts investigate and troubleshoot a Finding (e.g. EPSS and CISA KEV).

## Badges Claimed

The authors apply for the evaluation of the following badges:
* Available Artifacts (SeloD)
* Functional Artifacts (SeloF)
* Sustainable Artifacts (SeloS)

## Basic Information

**Hardware requirements:**

* A host with Docker installed.
* At least 8GiB of RAM to avoid issues, as the CVE metadata takes around 600MiB of RAM per uWSGI thread.

**Software requirements:**

* Docker and Docker Compose.
* Git.

**Repository structure:**

Our extensions introduce the following key additions to DefectDojo's upstream structure:
* `crivo/`: Contains scripts to initialize the environment and process external metadata.
* `crivo-model/`: Contains the vulnerability prioritization model integration.
* `docker-compose-crivo.yml`: Compose file for the GT-CRIVO initialization and alternate containers.
* `dojo/settings/crivo_settings.py`: Configuration file injected into DefectDojo settings.
* `risk_plugin/`: A new, separate Django app/plugin for the vulnerability assessment system, handling risk analysis and prioritization.

## Dependencies

* *CVE Metadata*: FIRST EPSS and CISA KEV feeds, downloaded automatically via the *crivo-init* container.

## Security Concerns

* *Default administrative credentials:* DefectDojo generates a random admin password on the first run, which is printed in the logs. If you expose this instance to an untrusted network, be sure to retrieve this password securely and change it if necessary.
* *Exposed ports:* DefectDojo exposes its web interface via Docker. Ensure your host firewall rules are properly configured.

## Installation

Follow these steps to clone the repository and launch the GT-CRIVO DefectDojo extensions:

```bash
# Clone the Git repo
git clone https://github.com/secure-net-systems/django-DefectDojo.git
# Checkout the extended branch
cd django-DefectDojo
git checkout crivo-rnp
# Build the Docker containers
docker compose build
# Run the crivo-init container to download CVE metadata, you
# only need to do this once:
docker compose -f docker-compose.yml -f docker-compose-crivo.yml run --rm crivo-init
# Launch the containers, pass the -d parameter if you want to detach
# container output from the terminal:
docker compose up -d
```

1. Wait for the system to come up.
2. The log messages will print out the administrator password. Search for the password in the logs:
    ```bash
    docker compose logs initializer | grep "Admin password:"
    ```
3. Log into the system interface at http://localhost:8080 with login *admin* and the generated password.
4. You should see the DefectDojo dashboard, confirming the application is running successfully.

## Minimal Test

After logging into the system, we can use it normally.  To test with an example
OpenVAS scan result and interact with the system, follow these steps:

* Create a new product
  * Click on Product > Add Product on the menu bar
  * Set "Test deployment" as the name and description
  * Choose "Research and Development" for the Product Type
  * Choose "No SLA Enforced" for SLA Configuration
* Load the product you just created
  * Click Product > All Products
  * Choose "Test deployment" from the table
* Load scan results
  * On the horizontal bar at the top, click Findings
  * In the drop-down, click "Import Scan Results"
  * Download [our example OpenVAS scan from](https://pugna.snes.dcc.ufmg.br/defectdojo/report-speed-lab.xml)
  * Set "Scan Type" to "OpenVAS Parser"
  * Load the OpenVAS file in the "Choose report file" at the bottom of the form
  * Set the "Group by" option to "Vulnerability ID from Tool"

The Findings and Finding Groups should now be browseable from the left sidebar.  The Finding view will show a "Risk" column that analysts can set after reviewing a vulnerability.  The detailed Finding view will show metadata for any Finding with one or more CVEs.

## Additional Documentation

More general documentation about the extensions is available in the [GT-CRIVO.md](GT-CRIVO.md) file.

# LICENSE

This project is an extension of DefectDojo. See the main repository's LICENSE file for details.

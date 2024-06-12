![basic kubernetes net](https://user-images.githubusercontent.com/8126042/113806145-a946db80-9716-11eb-82c3-129c16e3ef80.png)

# CP18-Cloud
Ongoing Project by Microsoft Software and Systems Academy Camp Pendleton Cohort 18 to design a decentralized private cloud.

## Current Development Targets:
- Peer to Peer VPN over Public IP - ZeroTier One
- K3s Hosted ZeroTier Controller with Azure-based etcd Database
- Redundant Docker Swarm for highly available application deployment
- GlusterFS distributed storage for Docker/K3s persistent storage

## Usage
The provided How-To guides will walk you through the steps required to build a redundant, HA container environment on Raspberry Pi's for a low-cost, easy to make testing environment

The current implementation makes use of an external Azure VM DB cluster for the K3s database, however this can be removed if a more efficient storage method is provided. The built in Pi storage via the MicroSD does not provide the required R/W speeds for an etcd database to function properly, and will result in reoccuring issues if attempted
### Real World Applications
Outside of the current usage for students as a test environment, the technologies employed in this project have clear applications to real-world scenarios inlcuding:

- HPC (High Performance Computing) - Cheap alternative for algorithm testing and data analysis
- CDN (Content Delivery Network) - Cheap, edge-cached storage for geographically distributed data access
- Point of Sale
- CCTV


## Storage
This implementation makes use of USB 3.0 thumb drives for Gluster storage, but this can just as easily be run using the builtin MicroSD storage as well.
### Contributors

- [Jameson Hearn](https://www.linkedin.com/in/jameson-hearn/ "Jameson Hearn")
- [Alexander Mcclain](https://www.linkedin.com/in/alexander-mcclain/ "Alexander Mcclain")
- [Scott Deflippo](https://www.linkedin.com/in/scott-defillippo/ "Scott Deflippo")
- [Brian Leyton](https://www.linkedin.com/in/brian-leyton/ "Brian Leyton")
- [Charles Ford](https://www.linkedin.com/in/charlesford1/ "Charles Ford")
- [Brianna Mikus](https://www.linkedin.com/in/brianna-mikus/ "Brianna Mikus")
- [Virgil Pua](https://www.linkedin.com/in/virgil-pua/ "Virgil Pua")

### GitHub Pull Request Guide:
- [GitHub How To](https://opensource.com/article/19/7/create-pull-request-github)

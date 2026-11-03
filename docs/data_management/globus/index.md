# Transferring Data with Globus

Globus is a fast, reliable tool for transferring data between institutions. 
Using the Globus web interface (Globus Online), you can start a transfer between any two institutions with servers connected to Globus. 
Once started, Globus manages the transfer for you and requires no further interaction, even if the transfer is interrupted.

To transfer data directly to or from your personal computer, install and run 
the [Globus Connect Personal software](https://www.globus.org/globus-connect-personal). 
Note that you cannot transfer files between two computers that are both running Globus Connect Personal.

For more information, visit [globus.org](https://www.globus.org/).

## Getting Started

### Log in to Globus

1. Go to [https://app.globus.org/file-manager](https://app.globus.org/file-manager).
2. Select **University of Iowa** from the organization drop-down list.
3. Log in with your HawkID on the University of Iowa login page.

If you already have a Globus account, you can use that instead.

## Accessing Data with Collections

Each source and destination for a transfer is called a **Collection**.

At the University of Iowa, ITS-Research Services operates a collection named <mark>**uiowadata#data**</mark>, which provides access to the following storage services:

| Storage Service                      | Path                        |
| ------------------------------------ | --------------------------- |
| Your Argon HPC home directory        | `/hpchomes/argon/<hawkid>/` |
| Dedicated Large Scale Storage shares | `/Dedicated/<sharename>/`   |
| Shared Large Scale Storage shares    | `/Shared/<sharename>/`      |
| NFS scratch filesystem               | `/nfsscratch/argon/`        |

!!! note "About automounts"

    These paths are automounts, so they aren't mounted or visible until accessed. Your directory may not appear when you browse, but if you enter the full path, it will mount on demand and work as though it had been there all along.

To transfer data to or from a personal computer, you will need to configure a private collection with the Globus Connect Personal software. See [Transfer data to your computer](#).

## Sharing Data

You can share data with collaborators by creating a **Guest Collection**. Not every storage service in the uiowadata#data collection supports this:

| Storage Service                      | Shareable via Guest Collection |
| ------------------------------------ | ------------------------------ |
| Your HPC home directory              | No                             |
| Dedicated Large Scale Storage shares | Yes                            |
| Shared Large Scale Storage shares    | Yes                            |
| NFS scratch filesystem               | No                             |

For step-by-step instructions, see [Sharing data](https://uiowa.atlassian.net/wiki/spaces/hpcdocs/pages/76550531/Sharing+data). You can also share a link to a single file rather than an entire folder; 
see [Get a shareable link for a file](https://uiowa.atlassian.net/wiki/spaces/hpcdocs/pages/76550505/Get+a+sharable+link+for+a+file).

To learn more about sharing with Globus, including how to assign roles that let others manage permissions on a collection you create, visit the [Globus sharing guide](https://docs.globus.org/how-to/share-files/).


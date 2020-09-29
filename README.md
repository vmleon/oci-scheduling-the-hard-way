# OCI Scheduling (The Hard Way)

Save some money when you don't need resources at out of business hours.

Learn some options of the Oracle Cloud Infrastructure (OCI) Command Line Interface (CLI) examples.

> USE THIS PROJECT AT YOUR OWN RISK
>
> THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY, FITNESS FOR A PARTICULAR PURPOSE, TITLE AND NON-INFRINGEMENT. IN NO EVENT SHALL THE COPYRIGHT HOLDERS OR ANYONE DISTRIBUTING THE SOFTWARE BE LIABLE FOR ANY DAMAGES OR OTHER LIABILITY, WHETHER IN CONTRACT, TORT OR OTHERWISE, ARISING FROM, OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE SOFTWARE.

## Create the virtual machine to be scheduled

Create a VM that we are going to use for testing the scheduling.

> For testing, use the smallest shape possible to save some credits

Make sure you know the Compartment the instance is being created in and get the OCID of that compartment in your notes.

Let's call the machine `scheduled`.

## Get familiar with OCI Command Line Interface

Get Compartment OCID from `Identity > Compartments`.

Open Cloud Shell:

```bash
export C=<compartment_OCID>
```

List your instances in the compartment and filter relevant information with jq:

```bash
oci compute instance list -c $C | jq '.data[] | {ocid: .id, name: ."display-name", status: ."lifecycle-state"}'
```

Export variable with instance OCID:

```bash
export INSTANCE_ID=<instance_ocid>
```

Ask nicely the instance to stop:

```bash
oci compute instance action --action SOFTSTOP --instance-id $INSTANCE_ID
```

As part of scripting for DevOps, it is nice if we wait for the action to be performed.

To Start the instance and wait for the action to be completed:

```bash
oci compute instance action --action START --wait-for-state RUNNING --instance-id $INSTANCE_ID
```

## Create the VM to schedule resources

Once again, create a new virtual machine. This time we are going to call it `scheduler`.

I recommend to pick the smallest shape as the workload is going to be minimal.

SSH into the machine

```bash
ssh opc@<public_ip>
```

Install the OCI CLI (Oracle Linux 8)

> For other versions, visit [OCI CLI installation](https://docs.cloud.oracle.com/en-us/iaas/Content/API/SDKDocs/cliinstall.htm)

```bash
bash -c "$(curl -L https://raw.githubusercontent.com/oracle/oci-cli/master/scripts/install/install.sh)"
```

Confirm all the default values

Confirm everything works with

```bash
oci -version
```

Configure the CLI to talk to Oracle Cloud

```bash
oci setup keys
```

The output should look like:

```
Public key written to: /home/opc/.oci/oci_api_key_public.pem
Private key written to: /home/opc/.oci/oci_api_key.pem
Public key fingerprint: aa:bb:cc:dd:ee:ff:00:11:22:33:44:55:66:77:88:99
```

Get the content of your public key

```bash
cat /home/opc/.oci/oci_api_key_public.pem
```

Copy the public key and go to your Oracle Cloud user settings on `Menu > Identity > Users > <your_user>`.

Then in the small menu on the bottom-left on `API Keys`

Click Add Public Key and `PASTE PUBLIC KEYS`, then paste the content of your public key

Confirm the fingerprint match the one created with `oci setup keys`.

Configure OCI CLI:

You can follow the steps in [Setting up the Config File](https://docs.cloud.oracle.com/en-us/iaas/Content/API/SDKDocs/cliinstall.htm#configfile)

```bash
oci setup config
```

> To retrieve the values [Where to Get the Tenancy's OCID and User's OCID](https://docs.cloud.oracle.com/en-us/iaas/Content/API/Concepts/apisigningkey.htm#five)

Press `ENTER` on `Enter a location for your config [/home/opc/.oci/config]:`

Set the User `OCID` on `Enter a user OCID:`

Set the Tenancy `OCID` on `Enter a tenancy OCID:`

Set the Region `name` on `Enter a region (e.g. ...`. I use `uk-london-1` but pick yours.

Answer `Y` to `Do you want to generate a new API Signing RSA key pair? (If you decline you will be asked to supply the path to an existing key.) [Y/n]:`

Default values for the rest.

Confirm with

```bash
oci os ns get
```

You should get a JSON response.

## Use a small VM to schedule other resources

Create a small VM on your Victual Cloud Network (VCN)

Set the instance as an instance principal with a Dynamic Group

Set up OCI CLI in the machine

Create start and stop bash scripts and test them

Create a `cron` tab for the schedule time.

## Watch out

This is just a simple example to make you familiar with OCI, in a production environment you should look out for:

- High Availability, what happens when the `scheduler` fail?
- Observability, do I know if the `scheduler` fails to perform an action?

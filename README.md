# OCI Scheduling (The Hard Way)

Save some money when you don't need resources at out of business hours.

Learn some options of the Oracle Cloud Infrastructure (OCI) Command Line Interface (CLI) examples.

> USE THIS PROJECT AT YOUR OWN RISK
>
> THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY, FITNESS FOR A PARTICULAR PURPOSE, TITLE AND NON-INFRINGEMENT. IN NO EVENT SHALL THE COPYRIGHT HOLDERS OR ANYONE DISTRIBUTING THE SOFTWARE BE LIABLE FOR ANY DAMAGES OR OTHER LIABILITY, WHETHER IN CONTRACT, TORT OR OTHERWISE, ARISING FROM, OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE SOFTWARE.

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

## Use a small VM to schedule other resources

Create a small VM on your Victual Cloud Network (VCN)

Set the instance as an instance pricipal with a Dynamic Group

Set up OCI CLI in the machine

Create start and stop bash scripts and test them

Create a `cron` tab for the schedule time.

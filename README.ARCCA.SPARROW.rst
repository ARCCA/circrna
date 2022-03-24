Run Circrna test on Hawk
------------------------
Create a directory to work on::

  mkdir ~/circrna-test
  cd ~/circrna-test

Load required files::

  module purge
  module use /apps/local/modules/projects
  module load nextflow/21.10.6
  module load scwXXXX/kubernetes
  module list

Clone ARCCA Circrna fork::

  git clone https://github.com/ARCCA/circrna.git

The ARCCA team is testing things on the sparrow branch. Confirm
you are in this branch if you want to see our current changes::

  cd circrna
  git branch

At this point you should be able to run Circrna pipeline test::

  nextflow kuberun -r sparrow -latest ARCCA/circrna -profile test,k8s

Access the data::

  nextflow kuberun login -v circrna-data:/data
  cd test_outdir
  ll
  exit

Copy data back to Hawk::

  kubectl cp ubuntu-new-and-improved:/data/hawk.username/test_outdir /scratch/hawk.username/output

Making changes (untested).
--------------------------
Assuming you have a GitHub account, create a fork. Go to https://github.com/ARCCA/circrna  and click on 'fork', follow the 
instructions to create the fork in your own repo.

- clone your fork repo on Hawk::

  git clone your-repo
  cd your-repo

- make changes, e.g.::

  vim main.nf
  git add main.nf
  git commit -m 'explain the changes made'
  git push

- run kubernetes cluster pointing to your own repo::

  nextflow kuberun -r your-branch -latest your-repo -profile test,k8s


Copying files between Hawk and Pods
-----------------------------------
In preparation to run a Nextflow pipeline it migth be necessary to
copy input files over to the Persistent Volume Claim (PVC). First you
need to find the name of a running pod::

  $ kubectl get pods
  NAME                                                 READY   STATUS    RESTARTS   AGE
  data-access                                          1/1     Running   0          16d
  friendly-wiles                                       1/1     Running   0          23d
  high-lamarck                                         1/1     Running   0          10d
  jovial-torvalds                                      1/1     Running   0          2d23h
  manila-csi-openstack-manila-csi-controllerplugin-0   3/3     Running   0          27d
  ...

In the above output we will use the `data-access` pod which has access to the PVC.
Then we can transfer files from Hawk to the PVC via the running pod::

  kubectl cp test_dir data-access:data/hawk-username/

To confirm that the data has been transferred we can use Nextflow login 
command to start a new pod and review the files. For this we need to
tell Nextfow the name of the PVC that should be mounted on the pod::

  $ kube get pvc
  NAME           STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS        AGE
  circrna-data   Bound    pvc-1ef84c5a-6301-4220-b09c-f110e59f0884   13970Gi    RWX            csi-manila-cephfs   27d

In this case the name of the PVC is `circrna-data` and we can now use Nextflow to
check what data was transferred::

  $ nextflow kuberun login -v circrna-data
  Pod started: distraught-keller
  bash-4.2# ls
  nextflow.config  results  test2  test4  test_dir  test_outdir  work
  bash-4.2# ll
  total 37
  -rw-r--r--   1 root   root    36204 Mar 24 14:44 nextflow.config
  drwxr-xr-x   3 root   root        1 Feb 25 14:13 results
  drwxrwxr-x   2 635422 9635422     0 Mar 24 14:44 test_dir
  drwxr-xr-x 257 root   root      255 Mar  7 17:22 work

The above command created and put you inside a pod called `distraught-keller`
(the name can vary) that gave access to check the contents of the PVC. After
we confirm that we have the desired data in place (in thi case our `test_dir`),
we logout with `exit` (this can take a couple of minutes while the pod is
termimated).

At this point we should be able to run our Nextflow pipeline pointing to the
location of the input files in the PVC `data/hawk-username/test_dir`.

If this work, then we can login again to check what files were produced and if 
everthing went as expected, exit and then copy any required files over to Hawk
(check quota available on scratch and the size of files that need to be 
transferred)::

  $ kubectl data-access:data/hawk-username/out_dir /scratch/hawk-username



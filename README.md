# EMR Cluster Setup Guide for Wine Quality Prediction

This guide provides step-by-step instructions for setting up an EMR (Elastic MapReduce) cluster on AWS to perform Wine Quality Prediction using Spark ML Libraries. The process includes setting up the EMR cluster, uploading files to S3, executing ML code on the cluster, and using Docker for execution.

## Creating EMR Cluster with Task Instance Groups

1. **Cluster Configuration**:
   - Create an EMR cluster with the following instance groups:
     - 1 Core Instance
     - 1 Master Instance
     - 4 Task Instances (for parallel processing)

2. **Key Pair**:
   - Generate a `.pem` key pair for SSH access to the cluster instances.

3. **IAM Roles**:
   - Configure IAM roles as specified in the AWS console.

## Accessing and Configuring EMR Cluster

1. **EC2 Console**:
   - Identify the master instance in the EC2 console.
   - Edit the inbound rules of the security group to allow SSH access from your IP.

2. **SSH Connection**:
   - Connect to the master node using SSH:
     ```bash
     chmod 400 EMR_Cluster.pem
     ssh -i EMR_Cluster.pem hadoop@ec2-xx-xx-xx-xx.compute-1.amazonaws.com
     ```

## Uploading Files to S3

1. **S3 Bucket**:
   - Create an S3 bucket named `pa2-ml-spark`.
   - Upload all required files (code, datasets, Dockerfile) to this bucket.

## Execution without Docker (Manual Setup) in instance

1. **Sync Files**:
   - Fetch code files from S3 to the EMR instance:
     ```bash
     sudo su  # Optional for root access
     aws s3 sync s3://pa2-ml-spark/ .
     ls  # Verify if all files are downloaded
     ```

2. **Install Dependencies**:
   - Install necessary Python dependencies:
     ```bash
     pip install numpy --user
     ```

3. **Run Training**:
   - Execute training script to generate a model:
     ```bash
     spark-submit training.py
     ```

4. **Hadoop Path Operations**:
   - Copy datasets to HDFS if needed:
     ```bash
     hadoop fs -copyFromLocal TrainingDataset.csv hdfs://ip-xx-xx-xx-xx.ec2.internal:8020/user/root/
     hadoop fs -copyFromLocal ValidationDataset.csv hdfs://ip-xx-xx-xx-xx.ec2.internal:8020/user/root/
     ```

## Execution with Docker from instance or anywhere else

1. **Build Docker Image**:
   - Log in to Docker and build the Docker image:
     ```bash
     docker login
     docker build -t sg2653/ml-spark .
     ```

2. **Run Docker Container**:
   - Execute the Docker container to run the ML code:
     ```bash
     docker run sg2653/ml-spark
     ```

3. **Push and Pull Docker Image**:
   - Push the Docker image to Docker Hub for future use:
     ```bash
     docker push sg2653/ml-spark
     ```
   - Pull the Docker image on any machine for execution:
     ```bash
     docker pull sg2653/ml-spark
     ```

## Downloading Files from EMR Instance

1. **Download Files**:
   - Use `scp` to download files from the EMR instance to a local machine:
     ```bash
     scp -i EMR_Cluster.pem -r 'hadoop@ec2-xx-xx-xx-xx.compute-1.amazonaws.com:/home/hadoop/*' /path/to/local_machine_folder
     ```

---

By following this guide, you will be able to set up and execute the Wine Quality Prediction ML model both manually on the EMR instance and using Docker for containerized execution. Ensure that all required files are uploaded to S3 and dependencies are properly installed for successful execution. Adjust paths and configurations as needed based on your specific setup.

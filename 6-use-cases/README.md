# 6. Data Security and Encryption Use Cases

In this section, we will demonstrate how to handle some of the common data security and encryption use cases with [Keyper](https://jarrid.xyz/keyper). These use cases are crucial for ensuring data security and compliance in modern applications. While steps 1-5 are not required, however they can be helpful if you encounter any issue.

## Whole File Encryption on Cloud Storage

Encrypting files stored in cloud storage is essential for protecting sensitive data at rest. Cloud security platforms like Dig or Wiz can identify data vulnerabilities, but encrypting the files ensures that even if unauthorized access occurs, the data remains unreadable. This tutorial will show you how to encrypt files in cloud storage and even automate the process within your existing tech stack.

➡️ [Go to Whole File Encryption on Cloud Storage Tutorial (AWS)](6-1-whole-file-encryption-on-cloud-storage-aws/README.md)  
➡️ [Go to Whole File Encryption on Cloud Storage Tutorial (GCP)](6-1-whole-file-encryption-on-cloud-storage-gcp/README.md)

## Sensitive Data Value Encryption

Encrypting individual data values is vital for protecting sensitive information within datasets. This is particularly useful when publishing records with encrypted fields (e.g., via Kafka, APIs) or encrypting raw values before inserting them into a database. By using [Keyper](https://jarrid.xyz/keyper), you can perform schema-aware encryption and decryption, ensuring that sensitive fields remain secure while allowing other fields to be used normally.

➡️ [Go to Sensitive Data Value Encryption Tutorial](6-2-sensitive-data-value-encryption/README.md)

## Encryption Key and Access Management - [WIP]

Simplifying access control across data platforms and infrastructure is critical for maintaining security and compliance. Instead of creating row/column-level permissions for each data store or platform individually, you can use encryption keys managed by [Keyper](https://jarrid.xyz/keyper) to control access. This approach streamlines the management of permissions and integrates seamlessly with IAM roles, reducing the complexity and potential for errors.

➡️ [Go to Encryption Key and Access Management Tutorial](6-3-encryption-key-and-access-management/README.md)

You are now able to handle various practical data security and encryption challenges. [Keyper](https://jarrid.xyz/keyper) provides the tools to implement robust security measures, ensuring your data remains protected and compliant with industry standards.

## Questions and Feedback

Thank you for following along with this tutorial series. If you have any questions, feel free to:

- [Reach out to us at Jarrid.](https://jarrid.xyz/#contact)
- [Ask questions on our discussion board.](https://github.com/orgs/jarrid-xyz/discussions)
- [Let us know if this was helpful to you?](https://tally.so/r/wMLEA8)

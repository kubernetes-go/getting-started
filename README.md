# Getting Started with Kubernetes

[![Website](https://img.shields.io/website?url=https%3A%2F%2Fdocs.kuberguide.org)](https://docs.kuberguide.org)
[![GitHub license](https://img.shields.io/github/license/kubernetes-go/getting-started)](https://github.com/kubernetes-go/getting-started/blob/main/LICENSE)
[![GitHub stars](https://img.shields.io/github/stars/kubernetes-go/getting-started)](https://github.com/kubernetes-go/getting-started/stargazers)

## Overview

Welcome to the **Getting Started with Kubernetes** repository! This repository contains the source code for [KuberGuide Documentation](https://docs.kuberguide.org/), a static documentation website built using the Hugo framework. Our goal is to provide comprehensive guides and tutorials to help users build and manage their own self-hosted Kubernetes clusters.

## About This Project

- **Website**: [docs.kuberguide.org](https://docs.kuberguide.org/)
- **Source Code**: [GitHub Repository](https://github.com/kubernetes-go/getting-started)

This project aims to make it easier for anyone to get started with Kubernetes by providing step-by-step instructions and detailed guides on setting up and managing a self-hosted Kubernetes cluster. We believe that learning Kubernetes from the ground up by building your own cluster can be an incredibly rewarding experience and can help deepen your understanding of this powerful orchestration platform.

## Features

- **Comprehensive Guides**: Detailed tutorials on setting up and managing a Kubernetes cluster.
- **Static Site**: Built with Hugo, making it fast, flexible, and easy to update.
- **Community Driven**: Contributions and feedback from the community are welcome.

## Getting Started

To get started with the documentation site, you'll need to have [Hugo](https://gohugo.io/getting-started/installing/) installed. Follow the steps below to clone the repository and run the site locally:

### Prerequisites

- Install Hugo: [Hugo Installation Guide](https://gohugo.io/getting-started/installing/)

### Clone the Repository

```bash
git clone https://github.com/kubernetes-go/getting-started.git
cd getting-started
git submodule update --init --recursive
cd ./guide
hugo server -D
```

Open your browser and visit `http://localhost:1313` to see the site in action.

## Contributing
We welcome contributions from the community! If you'd like to contribute, please fork the repository and use a feature branch. Pull requests are warmly welcome.

- Fork the repository
- Create a new branch (git checkout -b feature-branch)
- Make your changes
- Commit your changes (git commit -am 'Add some feature')
- Push to the branch (git push origin feature-branch)
- Create a new Pull Request

## License
This project is licensed under the MIT License - see the LICENSE file for details.

Thank you for visiting and happy learning!
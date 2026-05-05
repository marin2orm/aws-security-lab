# AWS Security Lab — Audit Prowler

Audit de sécurité d'un compte AWS réalisé avec [Prowler](https://github.com/prowler-cloud/prowler), outil open source utilisé en environnement professionnel pour l'analyse de conformité cloud.

---

## Contexte

Ce repository documente un audit de sécurité complet d'un compte AWS from scratch. L'objectif : identifier les mauvaises configurations, les comprendre, et les corriger — exactement ce qui est réalisé lors d'une mission d'audit cloud chez un client.

- Outil : Prowler v5.25.2
- Provider : AWS, région eu-west-3 (Paris)
- Date : Mai 2026
- Auteur : [marin2orm](https://github.com/marin2orm)

---

## Résultats du scan

587 checks exécutés. 129 fails (58.64%), 87 passed (39.55%).

Les frameworks couverts incluent SOC2, ISO 27001, CIS Benchmarks, GDPR, PCI-DSS, HIPAA, NIST et NIS2.

Le rapport HTML complet est disponible dans ce repository : `prowler-output.html`

---

## Analyse des fails critiques

### IAM — AdministratorAccess attaché directement à un utilisateur

Attacher une policy `AdministratorAccess` directement à un user IAM crée des credentials long-lived avec des droits illimités. En cas de compromission de la clé d'accès, un attaquant obtient un contrôle total du compte : suppression de ressources, exfiltration de données, désactivation des logs.

Remédiation : supprimer la policy du user, créer un rôle IAM avec des permissions limitées au strict nécessaire, utiliser AWS IAM Identity Center pour les accès admin temporaires, activer le MFA sur tous les users.

Frameworks concernés : SOC2, CIS 1.4 à 2.0, ISO 27001, PCI-DSS 4.0, NIST 800-53.

### IAM — MFA hardware non activé sur le compte root

Le compte root AWS a des droits absolus et ne peut pas être restreint par des policies IAM. Un MFA virtuel (application mobile) est moins sûr qu'un MFA hardware car il peut être compromis si l'appareil est perdu ou piraté.

Remédiation : acquérir une clé hardware MFA (YubiKey 5 NFC par exemple), désactiver le MFA virtuel actuel, ne jamais utiliser le compte root pour les opérations courantes.

Frameworks concernés : CIS 1.4 à 3.0, AWS Foundational Security Best Practices, SOC2, HIPAA.

### CloudTrail — journalisation non activée (17 fails High)

CloudTrail est le service de journalisation des appels API AWS. Sans CloudTrail activé sur toutes les régions, il est impossible de savoir qui a fait quoi sur le compte. C'est une exigence de base pour SOC2, ISO 27001 et PCI-DSS — sans logs, pas de certification possible.

```bash
aws cloudtrail create-trail \
  --name main-trail \
  --s3-bucket-name mon-bucket-logs \
  --is-multi-region-trail \
  --enable-log-file-validation

aws cloudtrail start-logging --name main-trail
```

---

## Comment reproduire cet audit

Prérequis : un compte AWS (Free Tier suffit), Python 3.10+, AWS CLI configuré.

```bash
# Installer Prowler
pip install prowler

# Lancer l'audit complet
python -m prowler aws

# Cibler uniquement IAM
python -m prowler aws --services iam

# Cibler un framework spécifique
python -m prowler aws --compliance cis_2.0_aws
```

---

## Ressources

- [Documentation Prowler](https://docs.prowler.com)
- [AWS Security Best Practices](https://aws.amazon.com/security/security-learning/)
- [CIS AWS Foundations Benchmark](https://www.cisecurity.org/benchmark/amazon_web_services)
- [AWS Well-Architected Framework — Security Pillar](https://docs.aws.amazon.com/wellarchitected/latest/security-pillar/welcome.html)

---

Marin — étudiant en cybersécurité cloud (BUT RT spécialité cybersécurité).  
Parcours orienté audit et conformité cloud AWS — objectif à terme : accompagner des entreprises dans leur mise en conformité SOC2, ISO 27001 et HDS.

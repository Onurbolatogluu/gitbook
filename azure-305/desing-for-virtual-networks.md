# 🪡 Desing for Virtual Networks

### Best Practices:

* **Adlandırma (Naming):** Sanal ağlarınız için benzersiz isimler oluşturmak üzere bir adlandırma kuralı geliştirmelisiniz. Sanal ağ isminin kaynak grubu içinde eşsiz olması önemlidir.
* **Bölgeler (Regions):** Sanal Ağlar bölgesel düzeyde oluşturulur ve bir kaynağı sanal ağa ilişkilendirmek için kaynağın da aynı bölgede bulunması gerekir.
* **Abonelikler (Subscriptions):** Her abonelik kapsamında mümkün olduğu kadar çok sanal ağ dağıtabilirsiniz.
* **Alt Ağlar (Subnets):** Adres alanını bölümlendirmek ve iş yüklerini izole etmek için NSG ve UDR’ler atamanız gerekmektedir. Azure Application Gateway, Azure Firewall, Azure Bastion gibi uygulamalar için özel alt ağlar planlayın.
* **Güvenlik (Security):** Gelen trafiği Azure Firewall, NVA, NSG gibi trafik filtreleme seçenekleri kullanarak filtreleyebilirsiniz. Tüm trafiği bu güvenlik duvarlarına zorlamak için özel yönlendirme tabloları kullanabilirsiniz.
* **Bağlantı (Connectivity):** Sanal ağlar arasında VNet Peering veya VPN Gateway kullanarak bağlantı planlayın. Daha geniş bağlantı seçenekleri için ER, S2S, P2S gibi hibrit bağlantı seçeneklerini de düşünün.
* **İzinler (Permissions):** Azure RBAC’i kullanarak sanal ağa erişimi kontrol edin. Sanal ağa özgü yerleşik rollerle yardım alabilir veya gerektiğinde özel roller geliştirebilirsiniz.
* **Politika (Policy):** Azure Policy, DDoS Koruması, akış logları gibi standartları denetleyip zorunlu kılacak politikalar içerir.


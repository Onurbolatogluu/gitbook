# 🦼 Design for storage security

Azure Storage güvenliğini sağlarken, Öncelikle, hesap anahtarları yerine, erişim süresi ve izinleri ince detaylarıyla belirleyebileceğiniz Shared Access Signatures (SAS) kullanmalısınız. Bu yöntem, verilere erişimi detaylı bir şekilde kontrol etmenize olanak tanır. Ayrıca, Azure Storage üzerindeki verilere yalnızca belirli ağlar veya IP adresleri üzerinden erişime izin vermek için firewall politikalarını etkinleştirerek güvenlik katmanını artırabilirsiniz. Erişimi kısıtlamak için, private endpoint kurarak güvenliği daha da güçlendirebilir ve bu sayede firmanızın local ağ altyapısını kullanarak Azure kaynaklarına erişim sağlayabilirsiniz.

Azure Storage'a yazılan verilerin şifrelenmesi için müşteri tarafından yönetilen anahtarları (CMK) kullanarak, şifreleme süreçlerinin üzerinde tam kontrol sahibi olabilirsiniz.

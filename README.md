# STM32 TIM2 1Hz Interrupt Blink ⏱️

Bu proje, **STM32F3 Discovery** geliştirme kartı üzerinde donanım zamanlayıcılarının (Timers) ve kesme (Interrupt) altyapısının temel çalışma prensibini doğrulamak amacıyla oluşturulmuştur. 

Sistem, `TIM2` donanım zamanlayıcısını kullanarak tam **1 saniyelik (1Hz)** hassas bir gecikme oluşturur ve her kesmede kart üzerindeki Mavi LED'in (PE8) durumunu tersine çevirir (Toggle).

## 🛠️ Kullanılan Donanım ve Araçlar
* **Geliştirme Kartı:** STM32F3 Discovery
* **Kütüphane:** HAL (Hardware Abstraction Layer)
* **Araçlar:** STM32CubeMX / STM32CubeIDE

## ⚙️ Sistem Konfigürasyonu ve Matematiksel Hesaplama

Projede HSE (High-Speed External) osilatör ve PLL kullanılarak sistem saati (SYSCLK) **72 MHz**'e ayarlanmıştır. 

Saniyede 1 kez (1Hz) kesme üretebilmek için `TIM2` konfigürasyonu aşağıdaki gibi yapılmıştır:
* **Clock (TIM_CLK):** 72 MHz
* **Prescaler (PSC):** 7199
* **Auto-Reload Register (ARR / Period):** 9999

**Frekans Hesaplama Formülü:**
$f_{Kesme} = \frac{f_{TIM\_clk}}{(Prescaler + 1) \times (Period + 1)}$

Değerler yerine konduğunda:
$f_{Kesme} = \frac{72000000}{(7199 + 1) \times (9999 + 1)} = 1 Hz$

Bu yapılandırma sayesinde, işlemciyi `HAL_Delay()` gibi "blocking" (engelleyici) fonksiyonlarla meşgul etmeden, donanım seviyesinde asenkron ve kusursuz bir zamanlama elde edilmiştir.

## 💻 Kritik Kod Bölümü

Zamanlayıcı süresi dolduğunda tetiklenen ve LED'i kontrol eden Geri Çağırım (Callback) fonksiyonu `main.c` içerisinde şu şekilde tanımlanmıştır:

```c
void HAL_TIM_PeriodElapsedCallback(TIM_HandleTypeDef *htim)
{
  // Kesmenin TIM2'den gelip gelmediğini kontrol et
  if (htim->Instance == TIM2)
  {
    // PE8 pinindeki Mavi LED'in durumunu tersine çevir (Toggle)
    HAL_GPIO_TogglePin(GPIOE, GPIO_PIN_8);
  }
}




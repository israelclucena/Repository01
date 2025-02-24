it('deve configurar o srcdoc do iframe e chamar os métodos necessários', () => {
  // Definindo valores para as variáveis necessárias
  component.url = 'http://example.com';
  component.ssoToken = 'token-exemplo';
  component.languageSelection = { value: 'en' };

  // Espionando o método sanitizer
  const sanitizeSpy = spyOn(component['sanitizer'], 'sanitize').and.callThrough();
  const bypassSecuritySpy = spyOn(component['sanitizer'], 'bypassSecurityTrustHtml').and.callThrough();
  
  // Espionando os métodos que devem ser chamados
  const watchIframeHeightChangesSpy = spyOn(component, 'watchIframeHeightChanges');
  const eventsAvoidSessionExpiredSpy = spyOn(component, 'eventsAvoidSessionExpired');

  // Criando um elemento iframe simulado
  const iframeElement = { nativeElement: { srcdoc: '' } };
  component.iframeElement = iframeElement as any;

  // Chama o método que vai configurar o srcdoc e chamar os outros métodos
  component.loadIframeContent();

  // Verifica se o sanitize foi chamado
  expect(sanitizeSpy).toHaveBeenCalled();

  // Verifica se o bypassSecurityTrustHtml foi chamado
  expect(bypassSecuritySpy).toHaveBeenCalled();

  // Verifica se o srcdoc foi configurado corretamente
  expect(iframeElement.nativeElement.srcdoc).toContain('<html>'); // Verifica se o srcdoc contém o conteúdo HTML

  // Verifica se os métodos watchIframeHeightChanges e eventsAvoidSessionExpired foram chamados
  expect(watchIframeHeightChangesSpy).toHaveBeenCalled();
  expect(eventsAvoidSessionExpiredSpy).toHaveBeenCalled();
});

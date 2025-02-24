it('deve configurar o srcdoc do iframe e chamar os métodos necessários', () => {
  // Definindo valores para as variáveis necessárias
  component.url = 'http://example.com';
  component.ssoToken = 'token-exemplo';
  component.languageSelection = { value: 'en' };

  // Mockando o método sanitize do sanitizer
  const sanitizeMock = jasmine.createSpy().and.returnValue('sanitizedContent');
  const bypassSecurityMock = jasmine.createSpy().and.returnValue('trustedHtml');
  
  component['sanitizer'].sanitize = sanitizeMock;
  component['sanitizer'].bypassSecurityTrustHtml = bypassSecurityMock;

  // Espionando os métodos que devem ser chamados
  const watchIframeHeightChangesSpy = spyOn(component, 'watchIframeHeightChanges');
  const eventsAvoidSessionExpiredSpy = spyOn(component, 'eventsAvoidSessionExpired');

  // Criando um elemento iframe simulado
  const iframeElement = { nativeElement: { srcdoc: '' } };
  component.iframeElement = iframeElement as any;

  // Chama o método que vai configurar o srcdoc e chamar os outros métodos
  component.loadIframeContent();

  // Verifica se o sanitize foi chamado
  expect(sanitizeMock).toHaveBeenCalled();

  // Verifica se o bypassSecurityTrustHtml foi chamado
  expect(bypassSecurityMock).toHaveBeenCalled();

  // Verifica se o srcdoc foi configurado corretamente
  expect(iframeElement.nativeElement.srcdoc).toContain('sanitizedContent'); // Verifica se o srcdoc contém o conteúdo "sanitizedContent"

  // Verifica se os métodos watchIframeHeightChanges e eventsAvoidSessionExpired foram chamados
  expect(watchIframeHeightChangesSpy).toHaveBeenCalled();
  expect(eventsAvoidSessionExpiredSpy).toHaveBeenCalled();
});

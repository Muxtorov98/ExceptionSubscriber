# ExceptionSubscriber

Create an event subscriber for handling exceptions in 
api/src/EventSubscriber/ExceptionSubscriber.php

````
<?php
declare(strict_types=1);

namespace App\EventSubscriber;

use Symfony\Component\EventDispatcher\EventSubscriberInterface;
use Symfony\Component\HttpFoundation\JsonResponse;
use Symfony\Component\HttpFoundation\Response;
use Symfony\Component\HttpKernel\Event\ExceptionEvent;
use Symfony\Component\HttpKernel\Exception\AccessDeniedHttpException;
use Symfony\Component\HttpKernel\KernelEvents;
use Symfony\Component\HttpKernel\Exception\HttpExceptionInterface;
use Throwable;

class ExceptionSubscriber implements EventSubscriberInterface
{
    public static function getSubscribedEvents(): array
    {
        return [
            KernelEvents::EXCEPTION => 'onKernelException',
        ];
    }

    public function onKernelException(ExceptionEvent $event): void
    {
        $exception = $event->getThrowable();

        switch (true) {
            case $exception instanceof AccessDeniedHttpException:
                $this->handleAccessDeniedHttpException($event);
                break;

            case $exception instanceof HttpExceptionInterface:
                $this->handleHttpException($event, $exception);
                break;

            default:
                $this->handleGenericException($event, $exception);
        }
    }

    private function handleAccessDeniedHttpException(ExceptionEvent $event): void
    {
        $response = new JsonResponse(
            $this->normalize('You do not have the required permissions.', 'Access Denied'),
            Response::HTTP_FORBIDDEN
        );

        $event->setResponse($response);
    }

    private function handleHttpException(ExceptionEvent $event, HttpExceptionInterface $exception): void
    {
        $response = new JsonResponse(
            $this->normalize($exception->getMessage()),
            $exception->getStatusCode()
        );

        $event->setResponse($response);
    }

    private function handleGenericException(ExceptionEvent $event, Throwable $exception): void
    {
        $response = new JsonResponse(
            $this->normalize($exception->getMessage()),
            Response::HTTP_INTERNAL_SERVER_ERROR
        );

        $event->setResponse($response);
    }

    private function normalize(string $description, string $title = 'An error occurred'): array
    {
        return [
            'title' => $title,
            'description' => $description,
        ];
    }
}
````

